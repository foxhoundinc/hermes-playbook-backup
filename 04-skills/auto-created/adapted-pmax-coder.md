# pmax-coder — Adapted Profile

**Profile ID:** pmax-coder  
**Base:** Mousa's original pmax-coder  
**Adapted:** 2026-04-15 — Privacy-focused removal of WhatsApp/email  
**Purpose:** Primary coding CLI with local Gemma inference  

---

## 🎯 Role

Primary coding agent for development tasks. Runs entirely locally with Gemma models. No API costs, no cloud dependencies.

## 🛠️ Configuration

```yaml
profile: pmax-coder
platforms: [cli]
models:
  primary: gemma4:26b-a4b-it-q5_K_M
  fallback: gemma4-31b-it-4bit
endpoints:
  primary: http://127.0.0.1:11434/v1
  fallback: ollama
```

## 🔧 Capabilities

- Code generation (Python, JavaScript, Go, etc.)
- CLI operations and shell scripting
- Local model inference only
- No external API calls

## 📋 Usage Examples

```bash
# Direct invocation via hermes CLI
hermes exec --profile pmax-coder -- "write a Python script that..."
```

## 🔄 Integration Notes

- Registered in `RESOLVER.md` as Always-on capability
- Triggers when task matches "coding" intent
- Bypasses OpenRouter to save costs

