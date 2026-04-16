# First Test Run

**Purpose:** Send real, live prompts through both machines to verify everything works end-to-end
**Run on:** Your phone/computer (Telegram or Discord)

---

## Prerequisites

All verification checks must pass first:
- ✅ NUC services running
- ✅ Win11 Hermes installed and configured
- ✅ LM Studio server running with models
- ✅ ACP connection working
- ✅ Syncthing verified

---

## Test 1 — Telegram Direct (NUC Gateway)

Send a message to your Telegram bot:

```
/ask What version of Hermes is running?
```

Expected response from NUC (via Telegram).

---

## Test 2 — Discord Direct (NUC Gateway)

Send a message in your Discord channel:

```
@Hermes what version?
```

Expected response from NUC (via Discord).

---

## Test 3 — Quick Local LLM (Win11)

From Telegram or Discord:

```
/ask --model lmstudio:qwen/qwen3.5-9b What is the capital of France?
```

Expected: Response via Win11's local LM Studio.

---

## Test 4 — Research Task (Win11 + Local LLM)

```
/ask --model lmstudio:qwen/qwen3.5-27b Summarize the key principles of deliberate practice from cognitive science research.
```

Expected: Longer response, should complete within 60 seconds.

---

## Test 5 — Vision Task (Win11 + Gemma)

```
/ask --model lmstudio:google/gemma-4-26b-a4b [attach an image] What is in this image?
```

Expected: Response from Win11 using Gemma 4 vision.

---

## Test 6 — Cloud Fallback (Win11 → OpenRouter)

If local models are slow or unavailable:

```
/ask --model qwen/qwen3.6-plus Give me a one-paragraph summary of the latest developments in AI agents.
```

Expected: Response from OpenRouter via Win11.

---

## Test 7 — Cron Job Test

```powershell
# On Win11 — trigger a test cron manually
hermes cron run --job-id SideHustle --dry-run
```

Expected: SideHustle job starts, generates output, delivers to Discord.

---

## Test 8 — Whisper Transcription (NUC)

```bash
# On NUC — test Whisper with a short audio clip
whisper ~/test_audio.mp3 --model base --language en

# Should produce:
~/test_audio.txt
```

If you have a real podcast to transcribe, run it through Whisper on NUC.

---

## Test 9 — File Sync (Syncthing)

```
/ask Summarize the current state of the Enhanced Mind project and save to ~/test_summary.txt
```

Then:
```bash
# On Win11 (WSL2):
cat ~/.hermes/test_summary.txt

# On NUC:
cat ~/HQ/HERMESHQ/test_summary.txt
```

Expected: File appears on both machines via Syncthing.

---

## Test 10 — ACP Delegation (NUC → Win11)

From NUC:

```bash
# Dispatch a heavy task to Win11
hermes acp dispatch \
    --to win11-heavy \
    --timeout 120 \
    --task "Generate a list of 10 cognitive science papers on skill acquisition"
```

Expected: Task runs on Win11, result returned to NUC.

---

## Test 11 — Full Daily Cycle

Simulate a full day of usage:

```
Morning (6AM cron fires):
  - AILearn cron → Win11 → research digest → Discord

During the day (manual):
  - You ask questions via Telegram
  - Local LLM handles quick lookups
  - Cloud handles complex research

Evening:
  - SideHustle cron → Win11 → side hustle research → Discord

Night:
  - NUC runs Whisper transcription on new podcasts
  - Syncthing keeps everything in sync
```

---

## Final Sign-Off

When all 11 tests pass:

```
===============================================
FIRST TEST RUN — COMPLETE
===============================================

[ ] Test 1 — Telegram (NUC)
[ ] Test 2 — Discord (NUC)
[ ] Test 3 — Local LLM quick
[ ] Test 4 — Local LLM research
[ ] Test 5 — Vision task
[ ] Test 6 — Cloud fallback
[ ] Test 7 — Cron job
[ ] Test 8 — Whisper
[ ] Test 9 — File sync
[ ] Test 10 — ACP delegation
[ ] Test 11 — Full daily cycle

Result: _________________ / 11

Date: _________________
Notes: _________________________________________

SETUP COMPLETE — SYSTEM IS OPERATIONAL
```

---

## Common First-Run Issues

| Issue | Fix |
|-------|-----|
| Telegram bot not responding | Check bot token, check NUC gateway logs |
| Discord bot not responding | Check bot token, check permissions |
| Local LLM timeout | Increase timeout, check LM Studio server |
| ACP dispatch fails | Check Win11 ACP is running, check firewall |
| Syncthing conflict | Resolve manually, check which machine edited |
| Cron not firing | Check Win11 cron config, check timezone |

---

## Documentation Complete

When everything is working, update the guide structure:

```
00-GUIDE-STRUCTURE.md — all items marked ✅ DONE
```

Your dual-system Hermes setup is now complete.
