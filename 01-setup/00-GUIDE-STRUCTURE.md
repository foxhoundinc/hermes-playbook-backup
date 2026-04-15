# Hermes Dual-System Setup Guide — Master Structure

**Purpose:** Complete setup guide for Win11 (heavy lifter + local LLM) + NUC (pure relay)
**Based on:** hermes-playbook v2026-04-14 + operational experience

---

## Section Index

| # | Section | Purpose | Status |
|---|---------|---------|--------|
| 01 | [Pre-Install](./01-pre-install/) | Backup verification, data inventory, pre-flight checks | ✅ Done |
| 02 | [Win11 Install](./02-win11-install/) | Full Hermes + LM Studio + local LLM setup | 📋 In Progress |
| 03 | [NUC Post-Install](./03-nuc-post-install/) | Optimize NUC to pure relay, strip bloat | 📋 Proposed |
| 04 | [Post-Install](./04-post-install/) | Both machines — Syncthing, Honcho, verification | 📋 Proposed |
| 05 | [Multi-Agent Setup](./05-multi-agent/) | Profile team: SOUL.md, AGENTS.md, team-agents.md | 📋 New |

---

## Detailed Breakdown

### 01-pre-install/
- `BACKUP-VERIFICATION.md` — Backup completed 2026-04-13 ✅
- `DATA-INVENTORY.md` — What we have, what's critical ✅
- `PRE-FLIGHT-WIN11.md` — Win11 hardware verification checklist ✅
- `PRE-FLIGHT-NUC.md` — NUC current state verification ✅

### 02-win11-install/
```
02-win11-install/
├── 01-prerequisites.md       # ✅ DONE — WSL2 (CachyOS/Arch) + Python + Git + Node.js + GPU passthrough
├── 02-lm-studio-setup.md     # ✅ DONE — LM Studio install + model downloads
├── 03-hermes-agent-install.md # ✅ DONE — Hermes clone + venv + dependencies
├── 04-acp-listener-setup.md  # ✅ DONE — ACP adapter for NUC→Win11 delegation
├── 05-config-priorities.md   # ✅ DONE — config.yaml for Win11
│                              # Includes: Native Google AI (Gemini) setup
│                              # Includes: Model stack (Qwen→MiniMax→Claude→Gemini)
├── 06-ssh-from-nuc.md        # ✅ DONE — NUC connects to Win11 as remote worker
└── 07-local-llm-guide.md     # ✅ DONE — Qwen3.5 + Gemma 4 models, how to run, when to use
```

### 03-nuc-post-install/
```
03-nuc-post-install/
├── 01-current-state-audit.md # ✅ DONE — What's running on NUC now
├── 02-strip-to-relay.md       # ✅ DONE — Remove anything not gateway/cron/syncthing
├── 03-ssh-hardening.md       # ✅ DONE — Key-based auth, firewall
├── 04-systemd-services.md    # ✅ DONE — Auto-start for gateway + Syncthing
├── 05-acp-worker-config.md   # ✅ DONE — Configure Win11 as remote worker in NUC config
└── 06-verification.md         # ✅ DONE — NUC relay working correctly
```

### 04-post-install/
```
04-post-install/
├── 01-syncthing-hardening.md  # ✅ DONE — Exclusions, verify HERMESHQ syncing
├── 02-honcho-integration.md   # ✅ DONE — Honcho already installed — verify it's working
├── 03-cron-relocation.md      # ✅ DONE — Which crons run where (NUC fires, Win11 executes?)
├── 04-verification-checklist.md # ✅ DONE — Full smoke test
└── 05-first-test-run.md       # ✅ DONE — Send test prompts, verify both machines respond
```

---

## Key Decisions Made (from playbook + notes synthesis)

### Architecture: How They Connect
```
User → Telegram/Discord → NUC (gateway, 24/7 relay)
                              ↓
                           Win11 (Hermes agent)
                              ↓
         ┌──────────────────┼──────────────────┐
         ↓                  ↓                  ↓
   LM Studio          OpenRouter          Native Google
   (Local GPU)        (Cloud API)         (Cloud API)
   Qwen2.5-14B/72B    Qwen→MiniMax→      Gemini 2.0 Flash
   Zero API cost      Claude Sonnet       Free tier
```

### Model Stack (6 Tiers) — UPDATED 2026-04-13
| Priority | Model | Provider | Use Case |
|----------|-------|----------|----------|
| 1 | Qwen3.6-plus | OpenRouter | Daily driver, free |
| 2 | MiniMax M2.7 | OpenRouter | Vision, fallback |
| 3 | Claude Sonnet 4 | OpenRouter | Best quality |
| 4 | Gemini 2.0 Flash | Native Google AI | Free backup |
| 5 | Qwen3.5-9B Q5 | LM Studio (local) | Fast daily tasks, ~7GB |
| 6 | Qwen3.5-35B-A3B MoE Q4 | LM Studio (local) | Deep research, ~21GB |
| + | Gemma 4-31B | LM Studio (local) | Vision + reasoning, ~19GB |
| + | Nous Chat (Hermes-4-70B) | Nous Portal | Free tier backup |

### Role Division
| Machine | Role | What It Runs |
|---------|------|-------------|
| Win11 | Heavy lifter | Hermes agent, LM Studio, all inference, heavy cron execution |
| NUC | Pure relay | Gateway only (Telegram + Discord), cron firing, Syncthing hub |

### Cron Strategy
- Win11 runs 24/7 (9950X idles at low power)
- Crons fire directly on Win11 (Option A)

### What Stays Local (Never Syncs)
- `~/.hermes/.env` — API keys
- `~/.hermes/config.yaml` — machine-specific
- LM Studio model cache (huge, Win11 only)

---

## Decisions Needed From You

### 1. Cron Execution ✅ DECIDED: Option A
Win11 runs 24/7, crons fire directly on Win11.

### 2. LM Studio vs Ollama ✅ DECIDED: LM Studio
Better NVIDIA GPU management, GUI makes it easier to manage.

### 3. Local Models ✅ DECIDED: Qwen3.5 + Gemma 4
Qwen3.5-9B (fast) + Qwen3.5-35B MoE (heavy) + Gemma 4-31B (vision)

### 4. Native Google AI Setup ✅ DECIDED: Included
Adding Gemini 2.0 Flash as standard guide step (free tier, 60 req/min, 1500/day)

### 5. Nous Portal ✅ INVESTIGATED
**Status:** Not as straightforward as expected.

**What Nous Portal Actually Offers:**
| Model | Free Tier | Paid Tier |
|-------|-----------|-----------|
| Hermes-3-Llama-3.1-70B | ✅ Free on Nous Chat | $0.40/1M tokens |
| Hermes-4-70B | ✅ Free on Nous Chat | $0.70/1M tokens |
| Hermes-4-405B | ✅ Free on Nous Chat | $1.50/1M tokens |
| Xiaomi MiMo v2 Pro | ❌ NOT FOUND | Not listed |

**Xiaomi MiMo v2 Pro** appears to have been a limited-time promotional offer that's no longer available.

**How to Access Free Tier:**
1. Go to https://chat.nousresearch.com
2. Create free account
3. Use Hermes-4-70B or Hermes-4-405B for free (limited usage)

**Recommendation:** 
- Not worth adding to the main model stack as a fallback
- But good to know about Nous Chat as an alternative access point
- **Removed from main stack** — your 4 tiers (Qwen → MiniMax → Claude → Gemini) are solid

---

## Recommended Model Stack for Your Setup

```
CLOUD (API):
├── Primary: Qwen3.6-plus (OpenRouter, free, 1M context)
├── Fallback 1: MiniMax M2.7 (OpenRouter, vision capable)
├── Fallback 2: Claude Sonnet 4 (OpenRouter, best quality)
└── Fallback 3: Gemini 2.0 Flash (Native Google AI, FREE tier)

LOCAL (Win11 + LM Studio):
├── Fast: Qwen3.5-9B Q5_K_M (GPU-only, ~7GB VRAM) ← Daily driver
├── Heavy: Qwen3.5-35B-A3B MoE Q4_K_M (offload to RAM, ~21GB) ← Deep research
└── Vision: Gemma 4-31B Q4_K_M (GPU, ~19GB) ← Multimodal reasoning
```

**Notre:** Nous Portal (Hermes-4-70B) available at https://chat.nousresearch.com as secondary free option.

This gives you:
- Zero API cost for bulk tasks (local)
- Best quality for complex work (Claude)
- Multiple free tiers (Qwen, Gemini)
- No single point of failure

---

## Sources Used
## Sources Incorporated

### Primary Sources
1. **hermesatlas.com** — Community ecosystem map (80+ projects, quality-filtered)
2. **State of Hermes April 2026 Report** — 57K stars in 6 weeks, v0.8.0 features
3. `../02-machines/optimal-setup.md` — Role division, config priorities
4. `../05-crons/cron-architecture.md` — Scheduling, resilience patterns
5. `HERMES-SETUP/Installation.md` — Win11 install steps (reference, needs update)
6. `HERMES-SETUP/Configuration.md` — Hermes config reference

### Operational Experience
- 3 weeks running Hermes on NUC (v0.40 → v0.80)
- Honcho already integrated
- Syncthing bridge working

---

## Critical Updates from Hermes Atlas / State of Hermes Report

### v0.90 New Features (April 2026)
|| Feature | What It Means for You |
|---------|----------------------|
| **`hermes backup --quick`** | New lightweight snapshot system (replaces raw tar/gzip) |
| **`watch_patterns` (Sentinel Mode)** | Real-time monitoring of terminal output or logs for regex/string triggers — proactive alerts for GPU temp spikes, crypto price thresholds, arbitrage signals, build failures |
| **Credential Pools** | Multi-API key rotation with automatic failover for 401/429 errors — eliminates rate limit dead stops across all providers |
| **Signal Gateway** | Encrypted messaging delivery for Web3 Sentinel alerts — Signal + Hermes for time-sensitive crypto ops |
| **Fast Mode (`/fast`)** | High-priority routing for OpenAI/Anthropic models — bypass standard wait times for time-sensitive tasks (trading, emergency fixes) |
| **Local Web Dashboard** | Browser-based GUI for sessions, skills, models — mobile-friendly, runs at `localhost:9119` |
| **Inline Diff Previews** | Visual "before and after" confirmation of file edits in the feed — catch errors before the agent commits |
| **Pluggable Memory** | Extensible provider interface for vector stores and custom DBs — next step beyond Honcho |
| **Camoufox Backend** | Stealth local browser for anti-detection web automation — advanced scraping on sites that block AI |
| **Termux Support** | Run Hermes natively on Android/Termux — "Pocket Brain" for 100% offline voice + image on mobile |
| **Native Google AI Studio** | Direct Gemini API — free tier (60 req/min, 1500/day) |
| **Live model switching** | Change models mid-session |
| **20+ LLM providers** | OpenRouter, Gemini, OpenAI, Anthropic, Nous Portal, local Ollama/LM Studio |
| **Memory providers** | 8 options (you use Honcho) |

### Hermes Atlas Ecosystem (Relevant to Your Setup)
| Category | Notable Projects | Stars |
|----------|------------------|-------|
| **Memory & Context** | Honcho (you have), Hindsight, autocontext | 9K, 731 |
| **Deployment** | llm-agents.nix, hermes-agent-docker | 1K, 8 |
| **Multi-Agent** | mission-control, hermes-agent-acp-skill | 4K, 7 |
| **Skills** | Wondelai (41 skills), Anthropic Cybersecurity (4.3K) | 543, 4.3K |

### Recommended New Additions to Consider
1. **hindsight** (9K stars) — Alternative vector memory for Enhanced Mind
2. **mission-control** (4K stars) — Multi-agent orchestration if you expand
3. **hermes-agent-self-evolution** (1.4K stars) — Self-improvement loop
4. **honcho-self-hosted** (142 stars) — If you want full Honcho control

---

*Last updated: 2026-04-14 — WSL2+CachyOS install path, v0.90 backup method*
