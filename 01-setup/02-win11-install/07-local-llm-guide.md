# Local LLM Guide

**Purpose:** How to use your local GGUF models on Win11 every day
**Hardware:** RTX 5070 Ti (16GB VRAM) + 64GB RAM
**Models:** LM Studio on `D:\LM Studio\models`

---

## Your Local Model Stack

| Model | Size | VRAM+RAM | Best For |
|-------|------|----------|----------|
| Qwen3.5-9B | 7GB | ~7GB | Fast daily driver, zero API cost |
| Qwen3.5-27B | 17GB | ~16GB+1GB | Research, long-form analysis |
| Gemma 4-26B-A4B | 17GB | ~16GB+1GB | Vision + reasoning, multimodal |

What does NOT fit your hardware:
- Qwen3.5-35B-A3B (21GB) — 3GB over your RAM ceiling
- Nemotron 120B (83GB) — needs more than your entire system

---

## Starting LM Studio

**GUI mode:**
- Start Menu -> LM Studio
- It minimizes to system tray
- Keep it running in the background

**Headless mode (for automation):**
```powershell
lms server start --model qwen/qwen3.5-9b --gpu-split auto
```

Server endpoint: `http://localhost:1234/v1`

---

## Switching Models

### In the LM Studio app
1. Click "Model" in sidebar
2. Click "Unload" on the current model
3. Search for a new model and click "Load"

### Via API (for scripts and cron jobs)
```bash
curl http://localhost:1234/v1/models
# Returns list of all downloaded models

# Load a specific model (if LM Studio supports it via API)
# Or just load it in the GUI before running your script
```

---

## When to Use Which Model

| Task | Model | Why |
|------|-------|-----|
| Quick lookups, formatting | Qwen3.5-9B | Fastest, GPU-only |
| Note summarization | Qwen3.5-9B | Zero API cost |
| Long document analysis | Qwen3.5-27B | Better quality |
| Research synthesis | Qwen3.5-27B | More context |
| Image analysis | Gemma 4-26B-A4B | Native vision |
| Vision + reasoning | Gemma 4-26B-A4B | Multimodal |

---

## In Hermes - Use Local via `/model` Override

In any chat, switch to a local model:

```
/model lmstudio:qwen/qwen3.5-9b
```

Then ask normally. Hermes routes to your local LM Studio server instead of cloud.

**In config** (`05-config-priorities.md`), set LM Studio as a fallback:
```yaml
providers:
  lmstudio:
    base_url: http://localhost:1234/v1
    api_key: local
```

---

## Batch Processing

### Example: Summarize research notes via local LLM
```bash
# 1. Have a text file
echo "Your research notes here" > notes.txt

# 2. Send to Win11 via SSH from NUC
scp notes.txt win11:C:\Users\<you>\notes.txt

# 3. Ask Hermes on Win11 using local model
hermes ask --model lmstudio:qwen/qwen3.5-27b "Summarize: $(cat C:\Users\<you>\notes.txt)"

# 4. Pull result back
scp win11:C:\Users\<you>\summary.txt ~/summaries/
```

---

## Memory Management

**GPU-only models** (Qwen3.5-9B): All layers stay in VRAM — fastest

**Offload models** (Qwen3.5-27B, Gemma 4-26B): Some layers spill to RAM — larger model fits but slower

### Monitoring VRAM
```powershell
nvidia-smi -l 1
```
Watch the memory column while a model is loaded.

### If You Run Out of VRAM
- Unload the current model before loading a new one
- Close other GPU apps (games, etc.)
- Use Q4 quantization instead of Q5 (30% less VRAM)
- Reduce context length in LM Studio model settings

---

## Performance Tips

| Tip | Impact |
|-----|--------|
| Keep context short | Faster, less VRAM |
| Unload idle models | Free VRAM for the next one |
| Q4 instead of Q5 | ~30% less VRAM, tiny quality loss |
| GPU-only when possible | 5-10x faster than offloading |
| Batch similar tasks | Don't switch models mid-session |

---

## API Quick Reference

Base URL: `http://localhost:1234/v1`

```bash
# List models
curl http://localhost:1234/v1/models

# Chat completion
curl http://localhost:1234/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen/qwen3.5-9b",
    "messages": [{"role": "user", "content": "Hello"}],
    "temperature": 0.7,
    "max_tokens": 500
  }'
```

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| Model won't load | Check VRAM+RAM usage; unload current model first |
| Server not responding | Verify LM Studio is running (check system tray) |
| GPU at 0% | Settings -> Hardware -> GPU Acceleration -> ON |
| Slow generation | Use GPU-only mode for small models; reduce context length |
| Context overflow | Use a smaller model or reduce context length |

---

## Cost Comparison

| Model | API Cost | Local Cost |
|-------|----------|------------|
| Qwen3.6-plus (cloud) | Free | - |
| MiniMax M2.7 | Pay per token | - |
| Claude Sonnet 4 | Pay per token | - |
| Qwen3.5-9B (local) | $0 | Electricity (~0.05/kWh) |
| Qwen3.5-27B (local) | $0 | Electricity |

Best practice: use local for all routine/automated tasks, cloud for one-off quality needs.
