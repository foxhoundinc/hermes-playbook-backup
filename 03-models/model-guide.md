# Local Model Guide — Qwen3.5 + Gemma 4
# LM Studio Setup for Win11

**Source:** lmstudio.ai/models (verified 2026-04-13)
**Purpose:** Which local GGUF models to download and when to use each one

---

## Qwen3.5 Family

| Model | Size | VRAM/RAM | Best For |
|-------|------|----------|----------|
| Qwen3.5-2B | 2GB | ~2GB | Embedded/low-power |
| Qwen3.5-4B | 3.75GB | ~4GB | Quick tasks, ultra-fast |
| **Qwen3.5-9B** | 7GB | ~7GB VRAM | **Daily driver — GPU-only, no offload** |
| **Qwen3.5-27B** | 17GB | ~16GB VRAM + 1GB RAM | **Research — tight VRAM fit** |

**Download links:**
- https://lmstudio.ai/models/qwen/qwen3.5-9b
- https://lmstudio.ai/models/qwen/qwen3.5-27b

---

## Gemma 4 Family

| Model | Size | VRAM/RAM | Best For |
|-------|------|----------|----------|
| Gemma 4-5.1B E2B | 4.2GB | ~4GB | Edge/low-power |
| Gemma 4-7.9B E4B | 5.9GB | ~6GB | Balanced |
| **Gemma 4-26B-A4B MoE** | 17GB | ~16GB VRAM + 1GB RAM | **Vision + reasoning — MoE efficient** |
| Gemma 4-31B (Dense) | 19GB | Must offload | Vision + reasoning, highest quality |

**Download links:**
- https://lmstudio.ai/models/google/gemma-4-26b-a4b
- https://lmstudio.ai/models/google/gemma-4-31b

---

## Recommended Stack for Your Setup

**Hardware:** Win11 — Ryzen 9 9950X, RTX 5070 Ti (16GB VRAM), 64GB DDR5 RAM

|| Task | Model | Size | VRAM+RAM | Notes |
||------|-------|------|----------|-------|
|| Fast daily tasks | **Qwen3.5-9B** | 7GB | ~7GB VRAM | GPU-only, no offload needed |
|| Research (balanced) | **Qwen3.5-27B** | 17GB | ~16GB VRAM + 1GB RAM | Tight VRAM fit, strong quality |
|| Vision + reasoning | **Gemma 4-26B-A4B MoE** | 17GB | ~16GB VRAM + 1GB RAM | 4B active params, vision native |
|| Light assistant | **Qwen3.5-4B** | 3.75GB | ~4GB VRAM | Ultra-fast, low-power tasks |

## Additional Cloud Models (from 0xJeff operational experience)

These models are validated by real-world use — add as cost-effective heavy-lifters via OpenRouter:

|| Model | Provider | Best For | Cost | Notes |
||-------|----------|----------|------|-------|
|| **Kimi k2.5** | OpenRouter | Long-running tasks | Low | Good token cost/efficiency ratio |
|| **GLM5.1** | OpenRouter | Long-running tasks | Low | Strong Kimi alternative |
|| **MiniMax 2.7** | OpenRouter | Conversational flow | Medium | Confirmed by 0xJeff — flows well |
|| **OpenCode Go ($5/mo)** | Nous Portal | Heavy task sessions | Flat $5 | Covers token costs vs raw API |

**Why these matter:** Qwen3.6-plus is your daily free driver, but for long-running research tasks, Kimi k2.5 and GLM5.1 offer better cost efficiency than Claude. MiniMax 2.7 confirmed as smooth for conversational work.

**Total local storage needed:** ~35GB (4 models)

**Does NOT fit your hardware:**
- Qwen3.5-35B-A3B MoE (21GB) — 3GB over your 64GB ceiling with VRAM offload
- Nemotron 3 Super 120B (83GB) — needs 83GB minimum RAM
- Qwen2.5-14B / Qwen2.5-72B — do not exist on LM Studio (only Qwen3.5 is available)

---

## When to Use Local vs Cloud

| Task | Use | Why |
|------|-----|-----|
| Quick lookups, formatting | Local Qwen3.5-9B | Fast, free, no API |
| Long-form writing, analysis | Local Qwen3.5-27B | Good quality, free |
| Image analysis (simple) | Local Gemma 4-26B-A4B | Vision native, MoE efficient |
| Vision + complex reasoning | OpenRouter MiniMax M2.7 | Best vision quality |
| Best quality text | OpenRouter Claude Sonnet 4 | Highest benchmark |
| Free fallback | Gemini 2.0 Flash | 60 req/min free |

---

## LM Studio Quick Setup

1. Download: https://lmstudio.ai/download
2. Install on Win11
3. Search for model name
4. Click Download
5. Load model in LM Studio
6. Enable "Local Server" for API access

**API endpoint:** `http://localhost:1234/v1/chat/completions`

---

## Links

- LM Studio: https://lmstudio.ai/models
- Qwen3.5: https://lmstudio.ai/models/qwen3.5
- Gemma 4: https://lmstudio.ai/models/gemma-4
