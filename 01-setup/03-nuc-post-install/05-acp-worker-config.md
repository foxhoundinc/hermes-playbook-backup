# ACP Worker Config — NUC Controls Win11

**Purpose:** Configure NUC's Hermes config so Win11 appears as a remote worker
**Run on:** NUC
**Based on:** `../02-win11-install/04-acp-listener-setup.md`

---

## Overview

```
NUC (control plane)                    Win11 (worker)
hermes-nuc                             hermes-win11
192.168.172.238                       192.168.x.x
                    ↕ ACP
           NUC dispatches tasks
           to Win11 for execution
```

---

## On NUC — Edit Hermes Config

```bash
# Edit NUC's Hermes config
nano ~/.hermes/config.yaml
```

Add the Win11 worker:

```yaml
# ============================================
# ACP Worker Configuration
# ============================================

acp:
  enabled: true

  # Local ACP listener (NUC listens on this)
  listener:
    host: 0.0.0.0
    port: 8080

  # Remote workers
  workers:
    # Win11 heavy worker (GPU + local LLM)
    - name: win11-heavy
      host: 192.168.x.x        # Win11 static IP
      port: 8080
      platform: windows
      auth: key                # SSH key auth
      timeout: 300             # 5 min timeout for heavy tasks

      capabilities:
        - local-llm            # Has LM Studio
        - gpu-inference        # RTX 5070 Ti
        - heavy-cron          # Can run heavy cron jobs

      limits:
        max_concurrent: 2      # Max 2 parallel tasks
        max_context: 32768     # Context window

    # NUC itself (local)
    - name: nac-local
      host: localhost
      port: 8080
      platform: linux
      capabilities:
        - gateway             # Telegram + Discord
        - cron               # Scheduling
        - whisper            # Audio transcription
        - light-tasks        # Quick tasks only
```

---

## On NUC — Validate Config

```bash
hermes config validate

# Should show:
# - ACP: enabled
# - Workers: 2 configured
# - win11-heavy: connected (or pending)
```

---

## Test Connection to Win11

```bash
# From NUC — ping Win11 ACP
hermes acp ping 192.168.x.x:8080

# Should return:
# Worker: win11-heavy
# Status: online
# Capabilities: local-llm, gpu-inference, heavy-cron
```

---

## Dispatch Tasks to Win11

```bash
# Send a task to Win11
hermes acp dispatch \
  --to win11-heavy \
  --task "transcribe /data/podcast.mp3"

# Send with higher timeout (for long tasks)
hermes acp dispatch \
  --to win11-heavy \
  --timeout 600 \
  --task "analyze research_notes.txt"

# View task status
hermes acp status

# Get result
hermes acp result --task-id <id>
```

---

## Set Default Worker Per Task Type

In `~/.hermes/config.yaml`:

```yaml
dispatch:
  defaults:
    - task_type: transcribe
      worker: nac-local          # Whisper runs on NUC

    - task_type: summarize
      worker: win11-heavy         # Local LLM on Win11

    - task_type: quick-edit
      worker: nac-local

    - task_type: research
      worker: win11-heavy         # Heavy LLM work on Win11
```

---

## Cron Job Dispatch

When NUC fires a cron, it can now delegate to Win11:

```bash
# Example: Daily research digest cron on NUC fires → delegates to Win11
# In crontab (on NUC):
0 6 * * * hermes acp dispatch --to win11-heavy --timeout 600 \
  --task "Run research digest: analyze new papers and summarize"

# Meanwhile NUC stays free for gateway + light tasks
```

---

## Verify Worker Health

```bash
# Check all workers
hermes acp status --all

# Expected output:
# Worker: nac-local
# Status: online
# Capabilities: gateway, cron, whisper, light-tasks
#
# Worker: win11-heavy
# Status: online
# Capabilities: local-llm, gpu-inference, heavy-cron
# Last seen: just now
# Load: 0/2 tasks
```

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| Worker offline | Check Win11 ACP service is running |
| Connection refused | Check Win11 firewall (port 8080) |
| Auth failed | Check SSH key auth from NUC to Win11 |
| Task times out | Increase timeout in config, or check Win11 load |
| Wrong worker used | Check dispatch.defaults in config |

---

## Next Step

After ACP is configured:
- **06-verification.md** — Full verification that everything works
