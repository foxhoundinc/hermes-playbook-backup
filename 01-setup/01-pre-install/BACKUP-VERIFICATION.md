# Pre-Install: Backup Verification & Data Inventory

**Purpose:** Before touching anything, verify all critical data is backed up and document current state.

**Completed:** 2026-04-13 08:52 AM

---

## Backup Status

| Data | Location | Backed Up | Synced to Win11 |
|------|----------|-----------|-----------------|
| Hermes config | `~/.hermes/config.yaml` | ✅ `nuc-critical-backup-20260413_085226/` | Via Syncthing |
| API keys | `~/.hermes/.env` | ✅ Same backup | Via Syncthing |
| Skills (3 weeks auto-improved) | `~/.hermes/skills/` | ✅ Same backup (17MB) | Via Syncthing |
| Chat sessions | `~/.hermes/sessions/` | ✅ Same backup (42MB) | Via Syncthing |
| Persistent memories | `~/.hermes/memories/` | ✅ Same backup (84KB) | Via Syncthing |
| Cron configs | `~/.hermes/cron/` | ✅ Same backup (64KB) | Via Syncthing |
| Honcho config | `~/.honcho/config.json` | ✅ Same backup | Via Syncthing |
| Channel directory | `~/.hermes/channel_directory.json` | ✅ Same backup (8KB) | Via Syncthing |
| hermes-agent code | `~/.hermes/hermes-agent/` | ❌ NOT backed up (6.2GB) | N/A — reinstallable |
| hermes-agent backup folder | `~/.hermes/backup/` | ❌ NOT backed up (6.2GB duplicate) | N/A — redundant |
| Cache/data | `~/.hermes/cache/` | ❌ NOT backed up | N/A — regeneratable |
| Logs | `~/.hermes/logs/` | ❌ NOT backed up | N/A — temporary |

**Backup location:** `~/HQ/HERMESHQ/BACKUPS/nuc-critical-backup-20260413_085226/`
**Backup size:** 127MB

---

## Data Inventory

### Hermes Core (~200MB critical)
```
~/.hermes/
├── config.yaml         # 9KB — current working config
├── .env                # 15KB — all API keys
├── channel_directory.json # 8KB — Discord/Telegram IDs
├── skills/             # 17MB — 41 Wondelai + auto-improved skills
├── sessions/           # 42MB — 3 weeks of chat history
├── memories/           # 84KB — persistent memory
├── cron/               # 64KB — cron job configs
└── checkpoints/        # 32KB — session checkpoints
```

### Honcho (~1KB)
```
~/.honcho/
└── config.json         # {enabled: true, memoryMode: hybrid}
```
**Note:** Honcho actual memory data is in the Hermes memory system, not a separate store.

### Syncthing Data (already synced ✅)
```
~/HQ/HERMESHQ/
├── hermes-playbook/    # This guide
├── HERMES-SETUP/       # Old install notes (reference only)
├── BACKUPS/            # New backup location
└── PROJECTS/           # Enhanced Mind data
```

---

## Pre-Install Checklist

Run this before starting any installation:

### On NUC — Verify Backup Integrity
```bash
# Check backup exists
ls ~/HQ/HERMESHQ/BACKUPS/nuc-critical-backup-20260413_085226/

# Verify critical files are present
cd ~/HQ/HERMESHQ/BACKUPS/nuc-critical-backup-20260413_085226/
ls -la config.yaml .env skills/ sessions/ memories/ cron/

# Check Syncthing status (should show "Up to Date")
syncthing-cli status
```

### On Win11 — Check Syncthing Received Backup
```powershell
# Verify backup folder synced
dir C:\HERMESHQ\BACKUPS\

# Should see: nuc-critical-backup-20260413_085226\
```

### On Win11 — Hardware Readiness
```powershell
# PRE-HERMES VERIFICATION
Write-Host "=== PRE-HERMES CHECK ===" -ForegroundColor Cyan

Write-Host "`n[1] Python:" -NoNewline
python --version

Write-Host "[2] Pip:" -NoNewline
pip --version

Write-Host "[3] Git:" -NoNewline
git --version

Write-Host "[4] Node.js:" -NoNewline
node --version

Write-Host "[5] CUDA + GPU:" -NoNewline
nvidia-smi --query-gpu=name,driver_version,memory.total --format=csv,noheader

Write-Host "[6] RAM:" -NoNewline
systeminfo | findstr /C:"Total Physical Memory"

Write-Host "`n=== If all show values, you're ready ===" -ForegroundColor Green
```

---

## Critical Notes Before Proceeding

1. **Win11 will be the inference powerhouse** — RTX 5070 Ti + 9950X + 64GB DDR5 can run 70B+ models locally
2. **NUC stays as pure relay** — 8GB RAM, i3 — no local inference, just gateway + crons + Syncthing
3. **This is NOT a fresh start** — we're optimizing 3 weeks of working setup
4. **API keys stay on both machines** — `.hermes/.env` stays local, never syncs
5. **Local LLM choice: LM Studio** — preferred over Ollama for GPU management on Windows

---

## Next Steps

- [ ] Verify backup on Win11 via Syncthing
- [ ] Run pre-Hermes verification on Win11
- [ ] Proceed to: `../02-win11-install/`
