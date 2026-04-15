---
name: model-registry
description: Track and manage all AI models used across Hermes Agent - cron jobs, auxiliary tasks, fallbacks, and credentials.
---

# Model Registry Management

Use this skill whenever model-related decisions are needed: selecting models for new jobs, diagnosing failures, auditing costs, or updating fallback chains.

## Current Model Chain (as of Apr 12, 2026)
```
Primary: qwen/qwen3.6-plus        (fast, free, 1M context)
Fallback 1: minimax/minimax-m2.7  (rate limit fallback, vision-capable)
Fallback 2: anthropic/claude-sonnet-4 (final fallback)
```
Vision tasks (image/video): Always route to MiniMax M2.7 — Qwen has no vision.

## Fallback Chain Logic
All cron jobs inherit the fallback chain from config.yaml. No health monitor needed — Hermes handles 429/503 automatically via the fallback chain.

## Cron Jobs
| Job | Model | Purpose |
|-----|-------|---------|
| SideHustle | Qwen3.6-plus | Daily side hustle research |
| AILearn | Qwen3.6-plus | AI learning digest |
| Scanner | Qwen3.6-plus / MiniMax | Free model scanning |

## Hardware VRAM/RAM Constraints (CRITICAL)
Your 5070 Ti (16GB VRAM) + 64GB system RAM = ~80GB total addressable for GGUF offloading. VRAM holds active layers; RAM holds overflow. Always check both.

**Critical:** When researching LM Studio models, always use browser navigation to https://lmstudio.ai/models — web_search/web_extract fail with "Payment Required". Model availability changes fast; old findings go stale.

| GPU | VRAM+RAM | Recommended Models (fits) | Does NOT fit |
|-----|----------|---------------------------|--------------|
| RTX 5070 Ti | 16GB VRAM + 48GB RAM | Qwen3.5-9B (7GB), Qwen3.5-27B (17GB), Gemma 4-26B-A4B (17GB) | Qwen3.5-35B-A3B (21GB), Nemotron 120B (83GB) |
| RTX 4090 | 24GB VRAM | All above + Qwen3.5-35B-A3B (21GB) | Nemotron 120B (83GB) |
| A100 | 40-80GB VRAM | Full models up to 70B | — |

**Known incompatible model:** `samuelcardillo/Carnice-MoE-35B-A3B-GGUF` — Hermes-optimized QLoRA fine-tune, but all quantizations require 24GB+ VRAM minimum (Q4_K_M = 20GB file but needs 24GB GPU). Does NOT fit 5070 Ti (16GB). Not viable for this setup.

## Free Model Scanning
Run `openrouter-free-model-scanner` skill periodically — free models appear and disappear. Always verify with browser navigation to model page (web tools fail on OpenRouter).

## Known Good Free Models (Apr 2026)
- **Gemma 4 31B IT (free)** — 262K context, text/vision, Apache 2.0
- **Gemma 4 26B A4B IT (free)** — 262K context, MoE efficient
- **Step 3.5 Flash (free)** — 256K context, text/reasoning

## When to Update
- A new cron job is created
- A job's model is changed
- A model fails and fallback is triggered
- Free scanner discovers a new free model worth noting
- Credentials added/removed from ~/.hermes/.env
- User asks about model usage or costs
