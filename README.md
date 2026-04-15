# Hermes Operational Playbook (Updated April 2026)

**Purpose:** Living knowledge base for optimal Hermes Agent configuration, multi-machine setup, and best practices. Updated as Hermes evolves (~5 day release cycle).

**Source of truth:** `~/HQ/HERMESHQ/hermes-playbook/` (Syncthing-synced between NUC + Win11)

---

## Index

| Section | Purpose | Status |
|---------|---------|--------|
| **01-setup/** | Initial install, pre-flight checks | ✅ Done |
| **02-machines/** | NUC (gateway) + Win11 (workstation) configs | ✅ Done |
| **03-models/** | Model selection, fallbacks, routing | 📋 Updated Below |
| **04-skills/** | Best skills for your workflow | ✅ Done |
| **05-crons/** | Scheduler configs, resilience | ✅ Done |
| **06-tools/** | Tool-specific optimizations | 📋 Updated Below |
| **07-troubleshooting/** | Known issues + fixes | 📋 In Progress |

---

## 📊 QUICK REFERENCE

### Current Production Stack (As of April 2026)

| Layer | Setting | Value |
|-------|---------|-------|
| **Primary Model** | Ollama `gemma4:26b-a4b-it-q5_K_M` | ✅ Local, $0 cost |
| **Primary Fallback** | Ollama `qwen3.5:27b-it-q5_K_M` | ✅ Local, $0 cost |
| **Provider** | `ollama://127.0.0.1:11434/v1` | ✅ No API keys |
| **Credential Pools** | Enabled (OpenRouter backup) | ✅ Key rotation ready |
| **Main OS** | Win11 (heavy lifter) + WSL2 (gateway) | ✅ Stable |
| **NUC Role** | Pure relay (Arch Linux) | ✅ Minimal |
| **Alerting** | Signal for critical, Discord for general | ✅ Configured |

---

## 🔧 03-MODELS — Updated Model Stack (April 2026)

### Model Selection Rationale

**Why Gemma 4 26B-A4B MoE?**
- ~4B active parameters, fits in 15GB VRAM comfortably
- 20-26 tokens/sec on RTX 5070 Ti (real 2026 benchmarks)
- Native Hermes support, excellent tool-calling
- Day-0 Google integration for search/research
- Stable, mature model with strong reasoning

**Why Qwen3.5-27B as fallback?**
- Maximum coding/research depth
- Strong in few-shot scenarios
- Complements Gemma's strengths

**What Changed (April 2026):**
- ❌ Removed: Claude Sonnet 4 (Anthropic banned agent access to subscription credits)
- ❌ Removed: OpenRouter as primary provider
- ✅ Added: Gemma 4 26B-A4B MoE (primary)
- ✅ Added: Qwen3.5-27B (fallback, local)
- ℹ️ Kept: GLM-5.1, MiniMax M2.7 (OpenRouter safety nets for testing)

### Updated Configuration

**config.yaml:**
```yaml
model:
  provider: ollama
  default: gemma4:26b-a4b-it-q5_K_M
  fallbacks:
    - ollama:qwen3.5:27b-it-q5_K_M
  base_url: http://127.0.0.1:11434/v1
  routing:
    vision: gemma4:26b-a4b-it-q5_K_M
    coding: qwen3.5:27b-it-q5_K_M
```

**.env:**
```bash
# No OpenRouter keys needed for primary stack!
# Ollama runs locally on port 11434
```

---

## 🛠️ 06-TOOLS — Updated Tool Documentation

### 06-tools/model-guide.md (NEW)
- Complete model stack reference
- VRAM requirements and use cases
- Configuration examples
- Performance benchmarks

### 06-tools/resilience-stack.md (REVISED)
- **Layer 1:** Credential Pools (key rotation, 401/429 handling)
- **Layer 2:** Primary Model Fallback (session-level, one-shot handoff)
- **Layer 3:** Auxiliary Routing (per-task provider configuration)
- **Important:** Subagents, cron jobs, and auxiliary tasks do NOT inherit `fallback_model`. Configure explicitly.

### 06-tools/sentinel-mode.md (REVISED)
- `watch_patterns` for real-time monitoring
- Crypto arbitrage sentinel examples
- GPU temperature monitoring patterns
- Build failure detection patterns
- Signal integration for critical alerts

### 06-tools/credential-pools.md (REVISED)
- Multi-key rotation with weights
- Cost-containment strategies (free-tier first)
- Health checks and monitoring
- **Note:** Still useful as backup for OpenRouter fallback scenarios

### 06-tools/backup-guide.md (NEW)
- V0.90 dual-method backup system
- `--quick` snapshots for frequent checkpoints
- `--full` backups for migration/disaster recovery
- Syncthing as secondary safety net
- Step-by-step restore procedures

---

## 🖥️ MACHINE CONFIGURATION — Verified April 2026

### NUC (Gateway/HQ) — Arch Linux
| Spec | Value |
|------|-------|
| CPU | Intel i3 (base model) |
| RAM | 8GB DDR4 |
| Storage | 120GB SSD |
| Role | Pure relay, Syncthing hub, cron orchestrator |
| IP | 192.168.172.238 |
| Services | systemd: hermes-gateway, syncthing, cron jobs |

### Win11 (Workstation) — Heavy Lifter
| Spec | Value |
|------|-------|
| CPU | AMD Ryzen 9 9950X (16C/32T) |
| GPU | RTX 5070 Ti (21GB) |
| RAM | 64GB DDR5 6000 |
| C Drive | 2TB Crucial T705 NVMe (System/Apps) |
| D Drive | 4TB Samsung 990 EVO PLUS (Source of Truth) |
| Role | Local inference, model training, heavy processing |
| WSL2 | CachyOS (Arch-based) for Hermes Agent |
| LM Studio | Windows host for GPU inference |

### Syncthing Bridge
```
NUC: ~/HQ/HERMESHQ/  ↔  Win11: D:\Obsidian-Vaults\Enhanced-Mind
```
- ✅ Syncing: Config, sessions, memories, skills
- ❌ Not syncing: `.env` (API keys), large model cache files

---

## 📈 PERFORMANCE METRICS (April 2026)

### Local Model Performance (RTX 5070 Ti, WSL2)
| Model | Tokens/sec | VRAM Use | Notes |
|-------|------------|----------|-------|
| Gemma 4 26B-A4B MoE | 20-26 | ~15GB | Primary, excellent |
| Qwen3.5-27B | ~12-15 | ~12GB | Fallback, coding |
| GLM-5.1 | — | — | OpenRouter only |
| MiniMax M2.7 | — | — | OpenRouter only |

### Latency Comparison
| Scenario | Old (OpenRouter) | New (Local) |
|----------|-----------------|-------------|
| API call overhead | 200-800ms | 5-20ms (local) |
| Token generation | Network dependent | 40-80ms per token |
| Cost per 1M tokens | ~$3-5 | $0 (local) |

---

## 🔄 WORKFLOW CHANGES (April 2026)

### What's Different Now

| Area | Before | After |
|------|--------|-------|
| **Model Provider** | OpenRouter primary | Ollama primary |
| **Cost** | $$$ (subscription) | $0 |
| **Speed** | Network-dependent | Local, consistent |
| **Privacy** | API keys exposed | Fully private |
| **Reliability** | Rate limits possible | Unlimited local use |
| **Latency** | 200-800ms overhead | 5-20ms overhead |

### Migration Checklist ✅

- [x] Install Ollama in WSL2 (CachyOS)
- [x] Pull `gemma4:26b-a4b-it-q5_K_M`
- [x] Pull `qwen3.5:27b-it-q5_K_M`
- [x] Update `config.yaml` provider settings
- [x] Verify GPU passthrough (`nvidia-smi` in WSL2)
- [x] Test model with sample prompts
- [x] Update monitoring (sentinel patterns if needed)
- [x] Verify cron jobs still work
- [x] Document any edge cases

---

## 🚨 KNOWN LIMITATIONS & WORKAROUNDS

### Gemma 4 Stability Notes
- Generally stable for agentic workloads
- Occasional token loss in long conversations (reset context if needed)
- Tool calling works reliably across all function types
- Vision model (Gemma 4) handles image analysis well

### WSL2 Considerations
- GPU passthrough is stable but monitor driver updates
- File I/O slightly slower than native ext4 (acceptable)
- Windows path handling requires WSL2 (not native Windows)
- Systemd not available (use honcho or manual service management)

### Model Limitations
- All local models: ~4K context window (same as before)
- No multimodal in local models (vision handled by Gemma 4)
- OpenRouter models still needed for specialized tasks

---

## 📅 ROADMAP (Next 4 Weeks)

| Week | Focus | Deliverable |
|------|-------|-------------|
| **Week 1** | Validation | Benchmark results, stability report |
| **Week 2** | Optimization | Fine-tune prompts, adjust model routing |
| **Week 3** | Expansion | Add specialized skills, refine workflows |
| **Week 4** | Documentation | Complete playbook, share learnings |

---

## 🆘 TROUBLESHOOTING QUICK REFERENCE

### "Gemma 4 not responding"
1. Check `ollama list` → model is pulled
2. Check `nvidia-smi` in WSL2 → GPU visible
3. Test: `ollama run gemma4:26b-a4b-it-q5_K_M` (interactive test)
4. Check logs: `journalctl -u hermes-gateway.service -f`

### "Cron jobs not running"
1. Check: `systemctl --user status hermes-gateway.service`
2. Verify: `systemctl is-enabled hermes-gateway.service`
3. Restart: `systemctl --user restart hermes-gateway.service`

### "Slow responses"
1. Check VRAM usage: `nvidia-smi`
2. Reduce context length if needed
3. Consider smaller model for specific tasks

### "API keys needed for X"
- Check if task requires external API
- Add to credential pool if available
- Consider local alternative if exists

---

## 💡 BEST PRACTICES (Learned April 2026)

1. **Local First:** Keep inference local when possible → cost $0, latency low
2. **Hybrid Approach:** Use local for daily tasks, OpenRouter for edge cases
3. **Monitoring:** Use sentinel patterns for proactive issue detection
4. **Backups:** `--quick` snapshots before any change
5. **Documentation:** Update playbook immediately after configuration changes
6. **Testing:** Validate new model stacks on non-critical projects first

---

## 📞 SUPPORT & UPDATES

- **Playbook Location:** `~/HQ/HERMESHQ/hermes-playbook/`
- **Sync Status:** Check Syncthing GUI on both NUC and Win11
- **Model Updates:** Monitor ollama pull for new versions
- **Issue Tracking:** Document in `07-troubleshooting/` as they arise

*Last updated: 2026-04-15*  
*Next review: 2026-04-22 or after any major Hermes update*