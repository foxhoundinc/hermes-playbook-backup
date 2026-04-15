# Adapted Profile Structure — Privacy-Focused (No WhatsApp)

**Purpose:** Profile definitions for actual usage (Telegram, Discord, Gemma native)
**Based on:** Mousa's original — adapted for privacy and local inference

---

## 🎯 YOUR ACTUAL PROFILES (6 Total)

### 1. pmax-coder — Primary Development CLI
- **Purpose:** Main coding agent, local Gemma inference
- **Platforms:** CLI only
- **Model:** `gemma4:26b-a4b-it-q5_K_M` (local, native)
- **Fallback:** `gemma4-31b-it-4bit` (MLX local)
- **Key Feature:** Zero API costs, fully offline

### 2. pmax-ops-observer — Health Monitoring
- **Purpose:** Daily system health reports
- **Platforms:** Aggregator (all platforms)
- **Model:** `qwen3.6-plus` (via OpenRouter for report generation)
- **Key Feature:** Monitors all services, sends summaries

### 3. pmax-content — Background Operations
- **Purpose:** Content generation, batch processing
- **Platforms:** Background service (no UI)
- **Model:** `gemma4` (local)
- **Key Feature:** Runs automated workflows

### 4. pmax-dareen — Content Creator Agent
- **Purpose:** Discord content creation and community management
- **Platforms:** Discord (primary), Telegram (support)
- **Model:** `gemma4:26b-a4b-it-q5_K_M` (local)
- **Key Feature:** Posts to Discord channels, manages community
- **Removed:** WhatsApp functionality (privacy)

### 5. pmax-tarek — Co-founder Communications
- **Purpose:** Team coordination, co-founder messaging
- **Platforms:** Telegram (primary), Discord (secondary)
- **Model:** `qwen3.6-plus` (OpenRouter for complex coordination)
- **Key Feature:** Handles team communication
- **Note:** Email not set up yet (add later if needed)

### 6. pmax-mousa — General Purpose Agent
- **Purpose:** Flexible user agent for ad-hoc tasks
- **Platforms:** Discord, Telegram
- **Model:** `qwen3.6-plus` (OpenRouter fallback)
- **Key Feature:** Adaptive, learns from usage patterns

---

## 🔧 PRIVACY-FOCUSED DESIGN

### What We Removed:
- ✗ WhatsApp (privacy concerns)
- ✗ Google Workspace dependencies
- ✗ Email account configurations
- ✗ Any WhatsApp API keys or webhooks

### What We Kept:
- ✓ Telegram (privacy-respecting alternative)
- ✓ Discord (open platform)
- ✓ Local Gemma models (zero cloud dependency)
- ✓ OpenRouter only where needed (non-WA costs)

---

## 📊 PLATFORM SUMMARY

| Profile | Primary Platform | Secondary | Model | Privacy Level |
|---------|-----------------|-----------|-------|---------------|
| pmax-coder | CLI | None | gemma4-local | 🔒🔒🔒🔒🔒 |
| pmax-ops-observer | Aggregator | All | qwen3.6-OJR | 🔒🔒🔒 |
| pmax-content | Background | None | gemma4-local | 🔒🔒🔒🔒🔒 |
| pmax-dareen | Discord | Telegram | gemma4-local | 🔒🔒🔒 |
| pmax-tarek | Telegram | Discord | qwen3.6-OJR | 🔒🔒🔒 |
| pmax-mousa | Discord | Telegram | qwen3.6-OJR | 🔒🔒🔒 |

*OJR = OpenRouter (used sparingly for complex tasks)*

---

## 🚀 GETTING STARTED

### For Your Clean Win11 Install:

```bash
# 1. Install Ollama with Gemma models
ollama pull gemma4:26b-a4b-it-q5_K_M
ollama pull gemma4-31b-it-4bit

# 2. Verify local installation
ollama list
# Should show both gemma4 models

# 3. Test Gemma native operation
hermes profiles test pmax-coder
```

### Telegram/Discord Setup:
- Add Telegram Bot token to config
- Invite Discord bot to channels
- Configure webhooks in `/etc/hermes/`

