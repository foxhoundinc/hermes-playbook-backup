# Hermes Backup & Restore Guide

**Purpose:** Document the V0.90 backup system, manage snapshots, and know when to use each method
**Applies to:** V0.90+ (Hermes Agent)
**Last updated:** 2026-04-15

---

## The Two Backup Methods

V0.90 introduces a **dual-method backup system**. Use the right one for the right situation.

|| Method | `--quick` (NEW in V0.90) | `--full` / `import` (Original) |
|---|---|---|
| **Command** | `hermes backup --quick --label "<label>"` | `hermes backup --full` or `hermes import /path/to/backup.zip --force` |
| **What it captures** | Selective state: `state.db`, `config.yaml`, `.env`, channel dir, gateway state | Everything in `~/.hermes/` — full clone |
| **Snapshot location** | `~/.hermes/state-snapshots/<id>/` | `~/.hermes/backups/<label>/` |
| **Size** | ~36-40MB (small — state.db is the bulk) | Full `.hermes/` tree (can be GB) |
| **Speed** | Fast | Slower |
| **Use when** | Pre-change checkpoint, quick before-upgrade snapshot, daily state safety net | Full system migration, complete clone to new machine, disaster recovery |
| **Restores** | Config + sessions + critical state | Full everything |

---

## Quick Snapshots — `hermes backup --quick`

**This is your default backup tool for V0.90.** Fast, lightweight, state-focused.

### Create a Quick Snapshot

```bash
# Before any major change — upgrades, config edits, skill changes
hermes backup --quick --label "pre-update-checkpoint"

# With a descriptive label (no spaces)
hermes backup --quick --label "v0.90-post-update"
```

Output:
```
State snapshot created: 20260414-024830-v0.90-post-update
  2 snapshot(s) stored in ~/.hermes/state-snapshots/
  Restore with: /snapshot restore 20260414-024830-v0.90-post-update
```

### List Snapshots

```bash
ls ~/.hermes/state-snapshots/
```

### What's In a Quick Snapshot

A `--quick` snapshot captures your **running state**, not your code or skills:

```
~/.hermes/state-snapshots/<id>/
├── state.db              # 36MB — SQLite with all sessions, memory, cron history
├── config.yaml           # Current working config
├── .env                  # API keys
├── channel_directory.json # Discord/Telegram ID mappings
├── gateway_state.json    # Active gateway session state
├── auth.json             # Provider auth tokens
├── processes.json        # Background process registry
└── manifest.json         # Snapshot metadata
```

**What's NOT included:** Skills (41 Wondelai + custom), Hermes source code, logs, cache. These are reconstructable or syncable.

### Restore a Quick Snapshot

**Use the gateway slash command (works in Telegram/Discord):**

```
/snapshot restore <snapshot-id>

# Example
/snapshot restore 20260414-024830-v0.90-post-update
```

Or restore using the CLI zip import (for full backup zips):

```bash
# Import a full backup zip
hermes import ~/hermes-backup-full-20260414_184634.zip --force
```

### Your Current Quick Snapshots

|| Snapshot ID | Label | Size | Date |
|---|---|---|---|---|
| 1 | `20260414-024830-v0.90-post-update` | v0.90-post-update | 36MB | 2026-04-14 02:48 AM |
| 2 | `20260414-085134-test-snapshot` | test-snapshot | 36MB | 2026-04-14 08:51 AM |

---

## Full Backups — `hermes backup` (no `--quick`)

**The nuclear option.** Creates a portable zip archive of everything in `~/.hermes/`.

### Create a Full Backup (timestamped)

```bash
# Creates: ~/hermes-backup-full-YYYYMMDD_HHMMSS.zip
hermes backup -o ~/hermes-backup-full-$(date +%Y%m%d_%H%M%S).zip
```

Output confirms:
```
Restore with: hermes import ~/hermes-backup-full-20260414_184634.zip
```

The zip contains your entire `~/.hermes/` tree — config, skills, sessions, memories, crons, everything.

### Restore a Full Backup

```bash
# Extract a previously created Hermes backup zip
hermes import ~/hermes-backup-full-YYYYMMDD_HHMMSS.zip --force
```

### When to Use Full Backup

- **Migrating to a new machine** — full clone to get everything running on day one
- **Before a major version jump** — e.g. before V0.91 if that's coming
- **Disaster recovery** — something goes wrong, restore everything at once
- **Infrequent** — not needed for every small change. Monthly or before major events is enough.

---

## Recommended Backup Schedule

|| Event | Method | Command |
|---|---|---|
| Before any config change | `--quick` | `hermes backup --quick --label "pre-config-edit"` |
| Before Hermes upgrade | `--quick` | `hermes backup --quick --label "pre-upgrade"` |
| Before installing new skills | `--quick` | `hermes backup --quick --label "pre-skills"` |
| Monthly full safety net | `--full` | `hermes backup --full --label "monthly-full-2026-04"` |
| Migrating to new machine | `--full` | `hermes export --all --output ~/HQ/HERMESHQ/BACKUPS/` |
| After major playbook changes | `--quick` | `hermes backup --quick --label "post-playbook-update"` |

---

## Syncthing as a Secondary Safety Net

Your NUC and Win11 are already Syncthing-synced. This means:

- **Config, sessions, memories** — live in `~/.hermes/` and Syncthing keeps a rolling copy on both machines
- **API keys (`.env`)** — NOT synced (local-only by design)
- **Skills** — synced via Syncthing to `~/HQ/HERMESHQ/.hermes/skills/`

Syncthing is **not a substitute** for snapshots — it tracks changes, not versions. But if your snapshot system fails, the Synced `.hermes/` tree on your NUC is a working fallback.

---

## Backup Before You Do Anything Else Right Now

```bash
# Right now — before you do the clean Win11 install
hermes backup --quick --label "pre-clean-install"
```

This captures your current working state (all sessions, memory, config, API keys, channel mappings). If anything goes wrong during the new install on Win11, you can restore to this point.

---

## If Disaster Strikes

**Hermes won't start after a config change:**
```
/snapshot restore <last-good-snapshot-id>
```

**Sessions missing after migration:**
```bash
hermes import ~/hermes-backup-<timestamp>.zip --force
```

**Config is corrupt:**
```bash
# Quick fix — restore just the config from your latest snapshot
cp ~/.hermes/state-snapshots/<id>/config.yaml ~/.hermes/config.yaml
```

**Lost API keys:**
```bash
# These are in your quick snapshot AND your .env was never synced to NUC
# Restore from the last snapshot
cp ~/.hermes/state-snapshots/<id>/.env ~/.hermes/.env
```

---

*Last updated: 2026-04-15*