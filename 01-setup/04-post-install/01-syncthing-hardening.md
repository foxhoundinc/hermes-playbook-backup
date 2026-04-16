# Syncthing Hardening

**Purpose:** Verify HERMESHQ syncs correctly between NUC and Win11, configure exclusions
**Run on:** NUC and Win11

---

## Current Syncthing Setup

```
NUC: ~/HQ/HERMESHQ/     ← Source of truth
Win11: D:\HermesHQ\   ← Synced clone (Syncthing target only)
```

Verify this is correct:
```bash
# On NUC — check Syncthing folder
syncthing-cli folder herameshQ

# Should show:
# Folder ID: herameshQ
# Path: /home/nac_username/HQ/HERMESHQ
# Devices: [win11-device-id]
# Status: Up to Date or Syncing
```

---

## Step 1 — Verify HERMESHQ Folder Exclusions

Syncthing config on NUC should exclude:
- `.DS_Store` and system files
- `node_modules/` (if any)
- Large temp files
- `papers_seen.txt` and cache files

On NUC:
```bash
nano ~/.config/syncthing/config.xml

# Or use the Syncthing web UI:
# http://localhost:8384 → Folder HERMESHQ → Edit → File Exclusions
```

Add to exclusions:

```
*.tmp
*.log
.DS_Store
Thumbs.db
desktop.ini
node_modules/
papers_seen.txt
papers_cache/
.chunks_tmp/
```

---

## Step 2 — Verify Syncthing Device IDs

Both machines must recognize each other.

```bash
# On NUC — get device ID
syncthing-cli devices

# Note the Device ID for Win11 (long alphanumeric string)

# On Win11 — get Syncthing device ID
# In Syncthing web UI: Actions → ID
# Or via CLI (if installed on Win11):
syncthing.exe -device-id
```

Verify each machine has the other's device ID configured:
- NUC should list Win11 as connected device
- Win11 should list NUC as connected device

---

## Step 3 — Verify File Sync

```bash
# On NUC — create a test file
echo "syncthing test $(date)" > ~/HQ/HERMESHQ/synctest.txt

# Wait 10-20 seconds

# On Win11 — check if it arrived
type D:\HermesHQ\synctest.txt

# Should contain "syncthing test" and today's date

# Clean up test file
rm ~/HQ/HERMESHQ/synctest.txt
# Verify it disappears on Win11 too
```

---

## Step 4 — Critical Files Never in Syncthing

These must NOT be synced (machine-specific):

| File | Why |
|------|-----|
| `~/.hermes/.env` | API keys — security risk |
| `~/.hermes/config.yaml` | Machine-specific paths |
| `~/.ssh/` | Private keys |
| `~/.npm/_cacache/` | Large cache |
| LM Studio model cache | 30GB+ — machine-specific |

Verify these are NOT in Syncthing by checking the folder configuration.

---

## Step 5 — Syncthing Service Auto-Start

Verify Syncthing starts on boot:

```bash
# On NUC:
systemctl --user is-enabled syncthing.service
# Should show: enabled

# On Win11:
# In Syncthing GUI: Settings → General → Start at boot → ON
```

---

## Step 6 — Stagger Sync Priorities

If HERMESHQ is large, prioritize what syncs first:

```bash
# On Win11 — pause all other folders
# In Syncthing GUI:
# Actions → Pause All

# Then manually trigger HERMESHQ sync
# Wait for it to complete
```

---

## Step 7 — Verify Hermes Home on Win11 (via WSL2)

Hermes runs inside WSL2 — access it via the Linux path:

```bash
# From WSL2 terminal — check Hermes home
ls -la ~/.hermes/

# Should have:
# - config.yaml
# - skills/
# - sessions/
# - memories/
# - crons/
```

---

## Syncthing Conflict Resolution

If two machines edit the same file at once:

```
Syncthing creates:
- notes.md (original)
- notes.md conflicts (NUC 2026-04-13 142305).syncthing-conflict
- notes.md conflicts (Win11 2026-04-13 142310).syncthing-conflict
```

**Rule:** Resolve conflicts manually — Hermes configs should never have conflicts because:
- NUC is the only one modifying configs
- Win11 only reads configs

---

## Full Syncthing Checklist

```
SYNCTHING VERIFICATION:
[ ] NUC: HERMESHQ folder configured
[ ] Win11: HERMESHQ folder configured
[ ] Both machines have each other's device IDs
[ ] Test file synced NUC → Win11
[ ] Test file synced Win11 → NUC
[ ] .env NOT in sync (local only)
[ ] config.yaml handled manually
[ ] LM Studio cache NOT in sync
[ ] Syncthing auto-starts on NUC boot
[ ] Syncthing auto-starts on Win11 boot
[ ] Syncthing web UI accessible on NUC (localhost:8384)
```

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| Files not syncing | Check device IDs, restart Syncthing |
| Syncthing won't start on boot | `loginctl enable-linger $USER` (NUC) |
| Out of space | `df -h` — clear cache, reduce retention |
| Conflict on critical file | Manually resolve, check timestamps |
| Slow sync | Check network, reduce folder size |

---

## Next Step

After Syncthing is verified:
- **02-honcho-integration.md** — Verify Honcho memory is working across sessions
