---
name: openrouter-free-model-scanner
description: Scan OpenRouter for free (max_price=0) AI models — new additions, context limit increases, and pricing changes. Uses browser fallback when API tools are blocked.
category: research
---

# OpenRouter Free Model Scanner

Scan openrouter.ai for free tier models with notable updates. Useful for cost optimization and finding new no-cost models.

## When to Use
- Finding free-tier AI models for cost optimization
- Discovering models with increased context windows
- Tracking new free model releases
- Comparing free vs paid model capabilities

## Steps

### 1. Navigate to free models page
```
https://openrouter.ai/models?max_price=0
```
Use browser navigation. Note: API tools (web_search, web_extract) often fail here due to Firecrawl paywalls.

### 2. Check for free models (filter by modality)
- Text only: `https://openrouter.ai/models?max_price=0&fmt=cards&input_modalities=text`
- Image/vision: `https://openrouter.ai/models?max_price=0&fmt=cards&input_modalities=image`
- Sort by newest: add `&sort=newest` to URL
- Sort by context length: add `&sort=context_length` to URL

### 3. Identify recent additions
Look for models with release dates within the past 7-14 days. The model list shows release date, context length, and pricing inline.

### 4. Get detailed specs on individual models
Navigate to each promising model's page directly:
```
https://openrouter.ai/{provider}/{model-name}
```
Example: `https://openrouter.ai/google/gemma-4-31b-it:free`

Key details to extract from the model page:
- **Context window** (shown near top of page)
- **Pricing** (input/output per 1M tokens)
- **Intelligence Index** and **Coding Index** from benchmarks section
- **Throughput** (tok/s) and **Latency** (s) from Performance tab
- **Tool Call Error Rate** from performance metrics
- **Cache Hit Rate** from effective pricing section

### 5. Cross-reference with rankings
```
https://openrouter.ai/rankings?max_price=0
```
The leaderboard shows which free models are gaining traction by volume.

## Key Free Models (as of April 2026)
- **Gemma 4 31B IT (free)** — 262K context, text/vision, Apache 2.0, intelligence 84th %ile, coding 90th %ile
- **Gemma 4 26B A4B IT (free)** — 262K context, text/vision/video, MoE efficient, Apache 2.0
- **Step 3.5 Flash (free)** — 256K context, text/reasoning, MoE 196B
- **Qwen3.6 Plus** — 1M context (paid, not free)

## Pitfalls
- web_search and web_extract fail with "Payment Required" on OpenRouter — always use browser tools
- The `sort=newest` parameter may not work in URL; rely on the "Newest" dropdown on the page
- Model pages don't always show full specs in compact snapshot; use full=true or navigate tabs for benchmarks
- Free tier status can change — always verify current pricing on the model page

## Verification
After finding a model, verify it's genuinely free by checking:
1. The model page header shows `$0/M input tokens $0/M output tokens`
2. The "Effective Pricing" section shows $0.0000 weighted average
3. The "Providers" section shows at least one provider with free routing
