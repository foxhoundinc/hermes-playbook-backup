# Recommended Setup for Your Hardware

**Source:** Orange Book §07 + gbrain skillpack + real-world patterns

---

## The Division of Labour

Given your hardware, here's the definitive split:

### NUC (8GB RAM / i3 / 120GB SSD) = Pure Relay
- Hermes gateway (Telegram + Discord listeners)
- Cron scheduler (fires jobs on schedule)
- Syncthing hub (HQ ↔ Win11)
- Home Assistant (if you want it)
- **Nothing else.** No local inference, no heavy tools.

### Win11 PC (9950X / 5070Ti / 64GB) = Heavy Lifter
- All cron job execution (Hermes agent runs here)
- Local GGUF model inference via llama.cpp (5070 Ti can run 70B models)
- Heavy file processing, media
- Development and testing
- Anything that needs serious CPU/GPU

### How they connect
```
Win11 PC (hermes-agent running) 
    ↕ SSH / shared filesystem
NUC (gateway + crons + syncthing)
    ↕ Telegram / Discord
User
```

---

## Installation Approach

### On the NUC
Fresh Arch install. Minimal — just enough to run:
- hermes-agent (lightweight CLI/client only, not heavy inference)
- hermes gateway (Telegram + Discord bots)
- syncthing
- systemd for auto-start

### On Win11 PC
Full hermes-agent install:
- `curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/install.sh | sh`
- Configure `~/.hermes/config.yaml` with your API keys
- Enable the toolsets you need

---

## Config Priorities

### 1. Model Config (current — working well)
```yaml
model:
  provider: openrouter
  api_key: sk-or-xxxxx
  model: qwen/qwen3.6-plus
  fallbacks:
    - minimax/minimax-m2.7
    - anthropic/claude-sonnet-4
```

### 2. Vision Routing
Qwen has no vision. Route all image tasks to MiniMax:
- Done via per-task model override in cron prompts
- Or via the vision_tool_diagnostics skill

### 3. Terminal Backend
```yaml
# For NUC (lightweight relay)
terminal: local

# For Win11 PC (heavy lifting)
terminal: local  # or docker if you want isolation
```

### 4. Gateway Config
```yaml
gateway:
  telegram:
    token: YOUR_BOT_TOKEN
  discord:
    token: YOUR_DISCORD_BOT_TOKEN
```

### 5. Toolsets
```yaml
toolsets:
  - web          # always on
  - terminal     # always on
  - file         # always on
  - skills       # always on
  - delegation   # for parallel research
  # Add per-task as needed:
  # - homeassistant
  # - mcp
```

### 6. MCP Servers (win11 only)
```yaml
mcp_servers:
  github:
    command: "npx"
    args: ["-y", "@modelcontextprotocol/server-github"]
    env:
      GITHUB_PERSONAL_ACCESS_TOKEN: "ghp_xxxxx"
    allowed_tools:
      - "list_issues"
      - "create_issue"
      - "get_pull_request"
```

---

## Workflow: Cron Jobs Run on Which Machine?

**Option A — NUC fires, Win11 executes:**
- Cron fires from NUC (always on)
- NUC SSH's to Win11 to run the agent
- Requires: SSH key auth from NUC → Win11

**Option B — Everything on Win11:**
- Win11 runs 24/7 (sleeps when not needed)
- Cron on Win11
- NUC just runs the gateway

**Recommendation: Option B.** The 9950X idles at low power anyway. NUC is overkill as a relay when Win11 can just sleep/wake.

---

## Syncthing Setup (already working)
```
NUC: ~/HQ/HERMESHQ/     ← source of truth
Win11: D:\Obsidian-Vaults\Enhanced-Mind\  ← synced clone
```

Known issue from your notes: Syncthing was dead on boot. Fix:
```bash
systemctl --user enable syncthing.service  # auto-start on boot
```

---

## Free Model Scanning
Use the `openrouter-free-model-scanner` skill weekly. New free models appear constantly. Your current chain (Qwen → MiniMax → Claude) is good but verify Qwen stays free.

---

## Local GGUF Inference (Win11 only)

With a 5070 Ti + 64GB RAM, you can run local models via **LM Studio**:

**Your local model stack (verified on lmstudio.ai 2026-04-13):**
- Qwen3.5-9B (7GB, GPU-only) — daily driver
- Qwen3.5-27B (17GB, VRAM+RAM offload) — research
- Gemma 4-26B-A4B MoE (17GB, VRAM+RAM offload) — vision

See `../03-models/MODEL-GUIDE.md` for full details.

```yaml
# config.yaml — local LM Studio
providers:
  lmstudio:
    base_url: http://localhost:1234/v1
    api_key: local
```

**Why bother:** Zero API cost, full privacy, no rate limits.
**Trade-off:** Slower than cloud, GPU VRAM limited.
