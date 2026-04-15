# pmax-tareen — Adapted Profile

**Profile ID:** pmax-tareen  
**Base:** Mousa's original pmax-tareen  
**Adapted:** 2026-04-15 — Privacy-focused configuration  
**Purpose:** Co-founder communication via Telegram and Discord  

---

## 🎯 Role

Team coordination agent handling co-founder messaging and collaboration.

## 🛠️ Configuration

```yaml
profile: pmax-tareen
platforms: [telegram, discord]
models:
  primary: qwen3.6-plus  # OpenRouter for complex coordination
  fallback: gemma4-local  # If OpenRouter unavailable
endpoints:
  telegram: your-bot-token-here
  discord: optional-bot-token
```

## 🔧 Capabilities

- Telegram messaging
- Discord notifications
- Team scheduling coordination
- OpenRouter-assisted planning

## 📋 Usage Examples

```bash
# Team coordination
hermes exec --profile pmax-tareen -- "notify co-founder about meeting"
```

## 🔄 Integration Notes

- Email not configured yet (can be added later)
- Telegram primary for real-time communication
- Discord as secondary channel
- OpenRouter used only for complex multi-step coordination

