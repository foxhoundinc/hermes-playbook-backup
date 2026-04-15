# Profile Setup — Create Your Team

**Purpose:** Create the 4-role profile team on Hermes
**Run on:** Machine where Hermes is installed

---

## Overview

You'll create 3 new profiles alongside the default Hermes profile:

| Profile | Role | Default Model |
|---------|------|---------------|
| hermes (default) | Orchestrator | Qwen3.6-plus |
| alan | Research | MiniMax M2.7 |
| mira | Writing | Gemma 4-31B |
| turing | Debugging | Qwen3.5-35B |

---

## Step 1: Verify Current Profile

```bash
hermes profile list
```

You should see at least the default profile.

---

## Step 2: Create New Profiles

```bash
# Create research profile
hermes profile create alan

# Create writing profile
hermes profile create mira

# Create debugging profile
hermes profile create turing
```

---

## Step 3: Verify Profiles Created

```bash
hermes profile list
```

Expected output:
```
Available profiles:
* hermes (default)
  alan
  mira
  turing
```

---

## Step 4: Add SOUL.md to Each Profile

Each profile needs a SOUL.md file for identity. See `02-SOUL-md-templates.md` for the full content to put in each.

```bash
# Location of profile SOUL.md files
~/.hermes/profiles/alan/SOUL.md
~/.hermes/profiles/mira/SOUL.md
~/.hermes/profiles/turing/SOUL.md
```

Create each file with the template from `02-SOUL-md-templates.md`.

---

## Step 5: Add Shared AGENTS.md

This file goes in the project directory, not a profile directory:

```
~/HQ/HERMESHQ/hermes-playbook/05-multi-agent/AGENTS.md
```

See `03-AGENTS-md.md` for the full content.

---

## Step 6: Configure Profile-Specific Models

Each profile can have its own default model in `config.yaml`:

```bash
# Edit alan's config
hermes -p alan config edit

# Set model
# model: minimax/minimax-m2.7
```

Recommended per-profile models:

| Profile | Model | Provider |
|---------|-------|----------|
| hermes | qwen/qwen3.6-plus | OpenRouter |
| alan | minimax/minimax-m2.7 | OpenRouter |
| mira | google/gemma-4-31b-it:free | OpenRouter |
| turing | qwen/qwen3.5-35b-a3b | LM Studio |

---

## Step 7: Test Each Profile

```bash
# Test alan (research)
hermes -p alan chat -q "What are the latest findings on cognitive load management?"

# Test mira (writing)
hermes -p mira chat -q "Write a brief summary of the Enhanced Mind project"

# Test turing (debugging)
hermes -p turing chat -q "Review this Python code: def foo(x): return x + 1"
```

---

## Step 8: Add Team Reference File

Create `team-agents.md` for documentation:

```
~/.hermes/team-agents.md
```

See `04-team-agents-md.md` for the full content.

---

## Telegram Integration

Each profile appears as a separate bot on Telegram. To route messages:

1. Create 4 separate Telegram bots (one per profile)
2. Configure each bot token in each profile's config.yaml
3. Each bot = one team member

See `hermes profile telegram-setup` for the bot creation workflow.

---

## Verification Checklist

| Check | Command | Expected |
|-------|---------|----------|
| Profiles exist | `hermes profile list` | 4 profiles shown |
| SOUL.md present | `cat ~/.hermes/profiles/alan/SOUL.md` | Identity text |
| AGENTS.md present | `cat ~/HQ/HERMESHQ/hermes-playbook/.../AGENTS.md` | Project context |
| Model configured | `hermes -p alan config get model` | minimax/minimax-m2.7 |
| Profile switch | `hermes -p mira chat --info` | Shows mira context |

---

## Next Step

See `02-SOUL-md-templates.md` for what to put in each profile's SOUL.md file.
