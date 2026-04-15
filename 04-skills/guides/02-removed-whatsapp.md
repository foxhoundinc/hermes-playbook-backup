# Removed WhatsApp Functionality — Privacy Decision

**Date:** 2026-04-15  
**Decision:** Remove all WhatsApp dependencies for privacy

---

## What Was Removed

- WhatsApp Business API integration
- WhatsApp webhook receivers
- WhatsApp template message configurations
- Any phone-number-based workflows

## Why This Was Removed

1. **Privacy-first approach** — No phone number tracking
2. **Reduced attack surface** — Fewer API keys to manage
3. **Simpler architecture** — Fewer moving parts
4. **Cost reduction** — No WhatsApp Business account needed

## What Replaced WhatsApp

| Original Use | Replacement |
|--------------|-------------|
| Customer notifications | Telegram bots |
| User support | Discord channels |
| Two-factor auth | Email (when added) or authenticator apps |

---

## Migration Notes

If you ever decide to add WhatsApp later:

1. Create new bot via WhatsApp Business API
2. Add webhook endpoint in `/etc/hermes/webhooks/whatsapp/`
3. Update profile configs to include WhatsApp platform
4. Add phone-number-to-user mapping in memory store

**But for now: No WhatsApp. Period.**

