# Honcho + Hindsight Integration

**Purpose:** Verify Honcho is installed and working on both machines, with Hindsight as alternative memory layer
**Based on:** Honcho memory setup skill + 0xJeff (2026-04-14) operational experience
**Run on:** NUC and Win11

---

## Memory Layer Options

Hermes supports two memory architectures:

| Provider | What it does | Best for | Status in your setup |
|----------|--------------|----------|---------------------|
| **Honcho** | Cross-session dialectical memory, 12 identity layers | Preferences, habits, patterns | ✅ Already installed |
| **Hindsight** | Vector-based session reflection, pattern recognition | Analyzing past sessions, relationship detection | 🆕 Add as complement |

0xJeff uses Hindsight as a reflection layer on top of Hermes sessions — it "reflects" on experiences and past sessions, allowing Hermes to analyze relationships and patterns that Honcho alone might miss.

**Recommended stack:** Honcho (primary, preferences) + Hindsight (secondary, session analysis). Both can run simultaneously.

---

## What Honcho Does

Honcho provides cross-session memory for Hermes Agent. It stores persistent facts about you (preferences, patterns, facts) that survive context compaction and session restarts.

---

## Check Honcho on NUC

```bash
# Is Honcho installed?
which honcho

# Version
honcho --version

# Check Honcho config
cat ~/.honcho/config.json

# Check memory store
ls -la ~/.honcho/memory/
```

Expected on NUC:
```
~/.honcho/
├── config.json      # Profile and memory config
└── memory/
    └── default/     # Default memory store
        └── memory.db  # SQLite or file-based store
```

---

## Honcho Config on NUC

Check `~/.honcho/config.json`:

```json
{
  "name": "Fox",
  "profiles": {
    "default": {
      "provider": "file",
      "path": "~/.honcho/memory/default"
    }
  },
  "model": {
    "provider": "openrouter",
    "model": "qwen/qwen3.6-plus"
  }
}
```

---

## Verify Honcho Memory

```bash
# Test Honcho by adding a fact
honcho add "Test fact from $(hostname) at $(date)"

# Retrieve it
honcho search "test fact"

# Should return the fact you just added

# Clear the test
honcho remove "test fact"
```

---

## Cross-Session Memory

The key test: does Honcho remember across sessions?

```bash
# Session 1 — add a fact
honcho add "Preferred model: Qwen3.6-plus free tier"

# Exit session

# Session 2 — retrieve it
honcho search "Preferred model"
# Should return "Preferred model: Qwen3.6-plus free tier"
```

---

## Syncing Honcho Between Machines

Honcho memory should live on the NUC (source of truth):

```
NUC: ~/.honcho/memory/    ← Source of truth
Win11: D:\HermesHome\.honcho\  ← Via Syncthing
```

If Win11 has its own Honcho:

```powershell
# On Win11 — configure Honcho to use Hermes home
# In PowerShell:
$env:HONCHO_HOME = "D:\HermesHome\.honcho"

# Or edit config.json
notepad D:\HermesHome\.honcho\config.json
```

---

## Connect Honcho to Hermes

In `~/.hermes/config.yaml`:

```yaml
memory:
  provider: honcho
  honcho:
    config_path: ~/.honcho/config.json
    # Memory auto-synced via Syncthing
```

On Win11, set HONCHO_HOME:
```powershell
# In PowerShell:
[System.Environment]::SetEnvironmentVariable(
    "HONCHO_HOME",
    "D:\HermesHome\.honcho",
    "User"
)
```

---

## Verify Honcho + Hermes Integration

```bash
# On NUC — check Hermes can reach Honcho
hermes memory status

# Should show:
# Provider: honcho
# Config: ~/.honcho/config.json
# Status: connected

# Test memory write/read
hermes memory add "Test integration $(date)"
hermes memory search "test integration"
```

---

## Honcho + Win11 Note

Win11's Hermes can use Honcho from Syncthing. No separate setup needed if:
1. Syncthing is syncing `~/.honcho/` from NUC
2. `HONCHO_HOME` env var on Win11 points to the synced location

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| `honcho: command not found` | `pip install honcho-mcp` or `npm install -g honcho` |
| Memory not persisting | Check disk space, check HONCHO_HOME |
| Win11 can't access memory | Check Syncthing sync, check HONCHO_HOME path |
| Config error | Validate JSON: `python -m json.tool ~/.honcho/config.json` |

---

## Full Honcho Checklist

```
HONCHO VERIFICATION:
[ ] NUC: honcho installed
[ ] NUC: config.json valid JSON
[ ] NUC: memory store exists and writable
[ ] NUC: add/retrieve test passed
[ ] NUC: cross-session persistence verified
[ ] Win11: HONCHO_HOME set to Syncthing path
[ ] Win11: Hermes sees honcho as memory provider
[ ] Both machines sharing same memory via Syncthing
```

---

## Next Step

After Honcho is verified:
- **03-cron-relocation.md** — Route cron jobs correctly between machines

---

## Hindsight — Alternative Memory Layer

**Source:** 0xJeff (2026-04-14)
**What it does:** Vector-based reflection on Hermes sessions — analyzes and explains relationships and patterns across past sessions
**Why add it:** Honcho captures what you said, Hindsight captures why things relate to each other

### Install Hindsight

```bash
pip install hindsight
# Or check the official repo for latest install
```

### Configure Hindsight + Hermes

In `~/.hermes/config.yaml`:

```yaml
memory:
  provider: honcho  # Primary — preferences and facts
  secondary:
    hindsight:
      enabled: true
      reflection_threshold: 7  # Auto-reflect after 7 sessions
```

### What Hindsight Adds

- **Pattern detection** — finds relationships between sessions Hermes couldn't see
- **Session reflection** — summarizes what happened across sessions and why
- **Corrections log** — tracks mistakes and learnings with explanation
- **Relationship mapping** — shows how concepts in your work connect over time

### Memory Layer Stack (Recommended)

```
Session Context (current)
        ↓
Honcho (who you are — preferences, facts, patterns)
        ↓
Hindsight (why things relate — reflection, analysis)
        ↓
Skills (how to do things)
```

This 3-layer stack means Hermes knows what you want, why topics connect, and how to execute.

### Verify Hindsight

```bash
# Check Hindsight is registered
hindsight --version

# Run a reflection scan
hindsight scan --sessions ~/.hermes/sessions/

# Check output for pattern insights
```

---

*Last updated: 2026-04-14 (0xJeff integration)*
