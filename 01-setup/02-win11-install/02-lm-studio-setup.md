# LM Studio Setup

**Purpose:** Install LM Studio on Windows 11, download local models, and enable the local API server
**Run on:** Windows 11 (native — NOT inside WSL2)
**Time needed:** 20-30 minutes + download time (~35GB)

---

## Architecture Note — LM Studio + WSL2

**LM Studio runs on Windows. Hermes runs inside WSL2 (CachyOS). They talk over `127.0.0.1:1234`.**

From inside WSL2, `127.0.0.1` is the same machine — no cross-network latency. Your RTX 5070 Ti inference is fully available inside WSL2.

Do NOT move LM Studio inside WSL2 — it needs direct GPU access on the Windows side.

---

## What We're Installing

LM Studio runs local GGUF models on your GPU. Your model setup:

| Model | Size | VRAM | Use |
|-------|------|------|-----|
| Qwen3.5-9B | 7GB | 7GB | Fast daily tasks |
| Qwen3.5-35B-A3B MoE | ~17GB | ~16GB + 1GB RAM offload | Research + heavy tasks |
| Gemma 4-31B | ~17GB | ~16GB + 1GB RAM offload | Vision + reasoning |

Total: ~35GB models on D: drive

---

## Step 1 - Download and Install LM Studio

1. Go to: https://lmstudio.ai/download
2. Download the Windows installer (.exe)
3. Run installer
4. Install to the **default location** (C:\Program Files\LM Studio)
5. Launch LM Studio from Start Menu

---

## Step 2 - Move Model Storage to D:

LM Studio defaults to storing models in your Windows user folder (on C:). Move the model storage to D: so the 35GB of models don't fill up C:.

1. In LM Studio: click the **Model** icon (sidebar, bottom)
2. Click the **folder icon** (top right of model panel)
3. Change the path to: `D:\LM Studio\models`
4. Create the folder if prompted: `D:\LM Studio\models`

That's it. Every model you download from now on goes to D:.

---

## Step 3 - Download Your Models

In LM Studio, click the **Search** icon (magnifying glass).

### Model 1 - Qwen3.5-9B (Daily Driver)
1. Search: `qwen3.5-9b`
2. Click the result from `qwen/qwen3.5-9b`
3. Click **Download** — choose `Q5_K_M` quantization
4. Wait for it to finish

### Model 2 - Qwen3.5-35B-A3B MoE (Research)
1. Search: `qwen3.5-35b-a3b`
2. Click `qwen/qwen3.5-35b-a3b`
3. Click **Download** — choose `Q4_K_M` quantization
4. Wait for it to finish

### Model 3 - Gemma 4-31B (Vision + Reasoning)
1. Search: `gemma-4-31b`
2. Click `google/gemma-4-31b`
3. Click **Download** — choose `Q4_K_M` quantization
4. Wait for it to finish

**Total download: ~35GB** — go make coffee.

---

## Step 4 - Enable Local Server

1. Click **Local Server** in the sidebar
2. Toggle **Enable Server** ON
3. Note the port: `1234`
4. The API endpoint will be: `http://127.0.0.1:1234/v1`

Test it (from Windows PowerShell):
```powershell
curl http://127.0.0.1:1234/v1/models
```

You should see JSON listing your downloaded models. From inside WSL2 (Ubuntu), the same `http://127.0.0.1:1234` address is reachable — that's how Hermes talks to LM Studio.

---

## Step 5 - Verify GPU is Being Used

1. Load a model (click the model in the left panel, then click **Load**)
2. Open Task Manager -> Performance -> GPU
3. While the model is loaded and responding, check GPU utilization

You should see GPU usage when generating. If GPU is at 0% and CPU is high, enable GPU acceleration:

- Settings -> Hardware -> GPU Acceleration -> ON

---

## Step 6 - Keep LM Studio Running

For Hermes to use local models, LM Studio needs to stay open in the background. It minimizes to the system tray.

To start LM Studio automatically:
1. Right-click the LM Studio icon in the system tray
2. Check "Start with Windows"

---

## Where Everything Lives

| What | Path | Size |
|------|------|------|
| LM Studio app | C:\Program Files\LM Studio | ~200MB |
| Models | D:\LM Studio\models | ~35GB |
| Settings | %LOCALAPPDATA%\LM Studio | <100MB |

---

## Quick Reference

**Start LM Studio:** Start Menu -> LM Studio
**Models endpoint:** `http://127.0.0.1:1234/v1/chat/completions`
**Check models (Windows):** `curl http://127.0.0.1:1234/v1/models`
**Check models (WSL2/CachyOS):** `curl http://127.0.0.1:1234/v1/models`

**In Hermes config:** Add `ollama` or `lmstudio` provider pointing to `http://127.0.0.1:1234/v1`

See **07-local-llm-guide.md** for how to use local models day-to-day.

---

## After This

Move to:
- **03-hermes-agent-install.md** - Install Hermes Agent