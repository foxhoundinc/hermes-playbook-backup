# Cron Relocation — Which Machine Runs What

**Purpose:** Finalize which crons run on which machine and configure the dispatch logic
**Based on:** `../05-crons/cron-architecture.md`
**Run on:** NUC and Win11

---

## Decision: Option A — Win11 Runs Its Own Crons

From your architecture decision: **Win11 runs 24/7, crons fire directly on Win11.**

```
Cron Architecture:
NUC: Gateway only (Telegram + Discord listeners)
       ↓
Win11: Fires and executes all cron jobs directly
       ↓
     ┌─────────────────────────────┐
     ↓                             ↓
Local LLM (Win11)           OpenRouter (fallback)
```

**Why this over Option B (NUC fires → delegates to Win11):**
- The 9950X idles at low power anyway
- Simpler — no ACP delegation overhead
- NUC stays light, Win11 handles everything
- ACP delegation still available for manual tasks when needed

---

## Current Cron Jobs

Based on your existing setup:

| Job | Schedule | Current Location | New Location |
|-----|----------|----------------|-------------|
| SideHustle | 3AM | NUC | Win11 |
| AILearn | 6AM | NUC | Win11 |
| Scanner | 7AM | NUC | Win11 |
| Whisper transcription | Manual/triggered | NUC | NUC (Whisper is CPU-only, stays on NUC) |

---

## Step 1 — Move Crons from NUC to Win11

On NUC, export current crons:
```bash
crontab -l > ~/crontab_backup.txt

# Then remove from NUC
crontab -r
```

On Win11, create the same crons:
```powershell
# Open Task Scheduler or use cron via WSL
# Or use Hermes cron system:
hermes cron create --name SideHustle --schedule "0 3 * * *" --prompt "..."
hermes cron create --name AILearn --schedule "0 6 * * *" --prompt "..."
hermes cron create --name Scanner --schedule "0 7 * * *" --prompt "..."
```

---

## Step 2 — Configure Cron on Win11

Using Hermes cron system on Win11:

```powershell
# Navigate to Hermes home
cd D:\HermesHome

# List current crons
hermes cron list

# Create SideHustle
hermes cron create `
    --name SideHustle `
    --model qwen/qwen3.6-plus `
    --schedule "0 3 * * *" `
    --prompt "You are a side hustle researcher..."

# Create AILearn
hermes cron create `
    --name AILearn `
    --model qwen/qwen3.6-plus `
    --schedule "0 6 * * *" `
    --prompt "You are an AI learning digest..."

# Create Scanner
hermes cron create `
    --name Scanner `
    --model qwen/qwen3.6-plus `
    --schedule "0 7 * * *" `
    --prompt "You are a free model scanner..."
```

---

## Step 3 — Whisper Stays on NUC

Audio transcription runs on NUC (CPU-based, no GPU needed):

```bash
# On NUC — Whisper transcription is triggered manually or via script
# Example trigger script:

#!/bin/bash
# /home/nac_username/scripts/transcribe.sh
# Usage: ./transcribe.sh /path/to/audio.mp3

AUDIO_FILE=$1
OUTPUT_DIR=~/HQ/HERMESHQ/transcripts/

whisper "$AUDIO_FILE" --model base --language en --output_dir "$OUTPUT_DIR"

# Notify Win11 via Hermes:
hermes tell @win11 "Transcription complete: $AUDIO_FILE"
```

---

## Step 4 — ACP for Manual Task Delegation

When you need a specific task on Win11 from NUC:

```bash
# From NUC — dispatch to Win11
hermes acp dispatch \
    --to win11-heavy \
    --timeout 300 \
    --task "Run this research: summarize the latest papers on [topic]"

# Check result
hermes acp status
```

This is optional — used only when you want NUC to trigger Win11 for specific tasks.

---

## Step 5 — Cron Monitoring

```powershell
# On Win11 — monitor cron health
hermes cron status

# View cron history
hermes cron history --limit 10

# If a cron fails:
hermes cron logs --job-id <id>
```

---

## Final Cron Flow

```
NUC (24/7):
  - Hermes Gateway (Telegram + Discord listeners)
  - Syncthing hub
  - Whisper transcription (CPU)
  - SSH access for admin

Win11 (24/7):
  - Hermes Agent (main execution)
  - All cron jobs (SideHustle, AILearn, Scanner)
  - LM Studio (local LLM inference)
  - OpenRouter API (cloud fallback)
  - Google AI (free tier fallback)
```

---

## Cron Checklist

```
CRON RELOCATION:
[ ] Crons exported from NUC
[ ] NUC crontab cleared
[ ] SideHustle cron created on Win11
[ ] AILearn cron created on Win11
[ ] Scanner cron created on Win11
[ ] Crons fire at correct times on Win11
[ ] Whisper stays on NUC (CPU task)
[ ] ACP delegation available for manual tasks
[ ] Cron history accessible on Win11
```

---

## Next Step

After cron relocation:
- **04-verification-checklist.md** — Full smoke test of both machines
