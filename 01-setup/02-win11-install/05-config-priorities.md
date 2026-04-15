# Config Priorities — Win11 Hermes Config

**Purpose:** Configure `~/.hermes/config.yaml` on Win11 with correct model stack and providers
**Based on:** `../../02-machines/optimal-setup.md` + `../../03-models/MODEL-GUIDE.md`
**Run on:** Windows 11 PC

---

## Full Config Template

Create or edit `C:\HermesHome\.hermes\config.yaml`:

```yaml
# ============================================
# Hermes Agent v0.80 — Win11 Heavy Lifter
# ============================================
# Hardware: Ryzen 9 9950X / RTX 5070 Ti / 64GB RAM
# Role: Heavy execution, local LLM inference, cron job execution
# ============================================

hermes:
  version: "0.80"
  home: C:\HermesHome

# --------------------------------------------
# MODEL CONFIGURATION — 6 Tiers
# --------------------------------------------
model:
  provider: openrouter
  model: qwen/qwen3.6-plus          # Tier 1: Daily driver (free, 1M context)
  fallbacks:
    - minimax/minimax-m2.7          # Tier 2: Vision + rate limit fallback
    - anthropic/claude-sonnet-4      # Tier 3: Best quality
    - google/gemma-4-31b-it:free    # Tier 4: Free Google model
  context_length: 32768

# Vision routing — Qwen has no vision, always use MiniMax for images
vision:
  default_provider: minimax
  default_model: minimax/minimax-m2.7

# --------------------------------------------
# LOCAL LLM — LM Studio (Win11 only)
# --------------------------------------------
providers:
  lmstudio:
    base_url: http://localhost:1234/v1
    api_key: local        # No key for localhost
    models:
      - name: qwen/qwen3.5-9b
        description: "Fast daily driver — 7GB, GPU-only"
        vrqam: 7GB
      - name: qwen/qwen3.5-27b
        description: "Research — 17GB, VRAM + RAM offload"
        vrqam: 17GB
      - name: google/gemma-4-26b-a4b
        description: "Vision + reasoning — 17GB, MoE"
        vram: 17GB
        vision: true

  openrouter:
    base_url: https://openrouter.ai/api/v1
    api_key: ${OPENROUTER_API_KEY}

  google_ai:
    provider: google_ai
    api_key: ${GOOGLE_AI_API_KEY}    # Free tier: 60 req/min, 1500/day

# --------------------------------------------
# LOCAL LLM USAGE RULES
# --------------------------------------------
local_llm:
  enabled: true
  preferred_provider: lmstudio
  auto_fallback_to_cloud: true      # If LM Studio is offline, use cloud

  # When to use local vs cloud
  use_local_when:
    - task: quick_lookups
      model: qwen/qwen3.5-9b
    - task: formatting_editing
      model: qwen/qwen3.5-9b
    - task: long_form_research
      model: qwen/qwen3.5-27b
    - task: image_analysis_simple
      model: google/gemma-4-26b-a4b

  use_cloud_when:
    - task: complex_vision
      model: minimax/minimax-m2.7
    - task: best_quality_text
      model: claude-sonnet-4
    - task: free_fallback
      model: google/gemma-4-31b-it:free

# --------------------------------------------
# TERMINAL
# --------------------------------------------
terminal:
  backend: local        # Windows Terminal, PowerShell 7+
  workdir: D:\HermesHome

# --------------------------------------------
# TOOLSETS
# --------------------------------------------
toolsets:
  - web           # always on — browsing, search
  - terminal      # always on — CLI tools
  - file          # always on — file operations
  - skills        # always on — skill system
  - delegation    # for spawning subagents
  - vision        # for image analysis
  # Optional (add as needed):
  # - homeassistant
  # - mcp

# --------------------------------------------
# GATEWAY — Telegram + Discord (NUC handles these)
# --------------------------------------------
gateway:
  telegram:
    enabled: false      # Keep off on Win11 — NUC handles this
    token: ${TELEGRAM_BOT_TOKEN}
  discord:
    enabled: false      # Keep off on Win11 — NUC handles this
    token: ${DISCORD_BOT_TOKEN}

# --------------------------------------------
# ACP — NUC-to-Win11 Delegation
# --------------------------------------------
acp:
  enabled: true
  host: 0.0.0.0
  port: 8080
  auth: key
  platform: windows
  workers:
    - name: win11-heavy
      host: 0.0.0.0
      platform: windows
      capabilities:
        - local-llm
        - gpu-inference
        - heavy-cron

# --------------------------------------------
# MEMORY — Honcho (cross-session)
# --------------------------------------------
memory:
  provider: honcho
  honcho:
    config_path: ~/.honcho/config.json

# --------------------------------------------
# CRON — Run on Win11
# --------------------------------------------
cron:
  enabled: true
  provider: local        # Win11 fires its own crons
  timezone: America/New_York    # Adjust to your timezone

# --------------------------------------------
# SYSTRAY (optional — run Hermes in background)
# --------------------------------------------
systray:
  enabled: true
  start_minimized: true
  show_notifications: true
```

---

## Environment Variables

Create `C:\HermesHome\.hermes\.env`:

```bash
# API Keys
OPENROUTER_API_KEY=sk-or-v1-xxxxx
GOOGLE_AI_API_KEY=AIzaSyxxxxx
TELEGRAM_BOT_TOKEN=123456:ABCxxxxx
DISCORD_BOT_TOKEN=xxxxx

# Hermes Home (if not set in system)
HERMES_HOME=D:\HermesHome
```

---

## Model Priority Summary

| Priority | Model | Provider | Use When |
|----------|-------|----------|----------|
| 1 | Qwen3.6-plus | OpenRouter | Daily driver, free |
| 2 | MiniMax M2.7 | OpenRouter | Vision, fallback |
| 3 | Claude Sonnet 4 | OpenRouter | Best quality needed |
| 4 | Gemini 2.0 Flash | Google AI (free) | Rate limited / free backup |
| 5 | Qwen3.5-9B | LM Studio (local) | Quick tasks, zero API cost |
| 6 | Qwen3.5-27B | LM Studio (local) | Research, zero API cost |
| 7 | Gemma 4-26B-A4B | LM Studio (local) | Vision + reasoning |

---

## Applying Config

```powershell
# After saving config.yaml:
hermes config validate

# Restart Hermes to apply:
hermes gateway restart

# Or for CLI-only changes:
hermes restart
```

---

## Notes

- LM Studio must be running for local models to work (unless `auto_fallback_to_cloud: true`)
- Gateway for Telegram/Discord stays OFF on Win11 — NUC handles all gateway traffic
- Cron jobs on Win11: run directly, no SSH needed from NUC
- If you want NUC to trigger jobs on Win11, use ACP (see 04-acp-listener-setup.md)

---

## Next Step

After config is set:
- **07-local-llm-guide.md** — Detailed guide for using local models day-to-day
- **../04-post-install/03-cron-relocation.md** — Which crons go where
