# pmax-dareen — Adapted Profile

**Profile ID:** pmax-dareen  
**Base:** Mousa's original pmax-dareen  
**Adapted:** 2026-04-15 — Privacy-focused removal of WhatsApp  
**Purpose:** Discord content creation and community management  

---

## 🎯 Role

Content creator agent focused on Discord-based content publishing and community engagement.

## 🛠️ Configuration

```yaml
profile: pmax-dareen
platforms: [discord, telegram]  # Telegram as backup
models:
  primary: gemma4:26b-a4b-it-q5_K_M
  fallback: gemma4-31b-it-4bit
endpoints:
  discord: your-bot-token-here
  telegram: optional-bot-token
```

## 🔧 Capabilities

- Discord message posting
- Community moderation assistance
- Content scheduling
- Local Gemma inference

## 📋 Usage Examples

```bash
# Invoke for content creation
hermes exec --profile pmax-dareen -- "create a post about our new feature"
```

## 🔄 Integration Notes

- Removed all WhatsApp dependencies
- Uses local Gemma for zero-cost inference
- Can post to multiple Discord channels
- Telegram used only for fallback notifications

