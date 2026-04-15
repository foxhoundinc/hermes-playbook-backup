# AGENTS.md — Shared Project Context

**Location:** `~/HQ/HERMESHQ/hermes-playbook/05-multi-agent/AGENTS.md`
**Purpose:** Project knowledge shared across all profiles (not identity)

---

## Project: Enhanced Mind

**Goal:** Cognitive optimization platform using elite performance data

### Core Research Areas
- core-theory/ — Foundational cognitive frameworks
- elite-performance/ — Peak performer strategies
- neuroscience/ — Brain science underlying cognition
- skill-acquisition/ — How to learn skills faster

### Key Sources (Processed)
- Tim Ferriss podcasts (129+ transcripts)
- Huberman Lab episodes
- Joe Rogan Experience episodes
- Regenerative Performance audiobook
- The Core (athletic development)
- The Stimulated Mind

### Discord Channels
- #core-theory
- #elite-performance
- #neuroscience
- #skill-acquisition
- #platform-architecture
- #research-log
- #citation
- #deep-dives

---

## Project Structure

```
~/HQ/HERMESHQ/                    # Syncthing root (NUC)
├── hermes-playbook/              # This playbook
│   ├── 01-setup/                 # Install guides
│   ├── 02-machines/              # Machine configs
│   ├── 03-models/                # Model selection
│   ├── 04-skills/                # Skills library
│   ├── 05-multi-agent/           # Profile team ← YOU ARE HERE
│   └── 06-tools/                 # Tool guides
├── .chunks_tmp/                  # Processing temp
└── [other project folders]
```

On Win11: `D:\Obsidian-Vaults\Enhanced-Mind\` (synced clone)

---

## Coding Conventions

| Language | Style |
|----------|-------|
| Python | PEP 8, type hints where helpful |
| Bash | Shellcheck validated |
| Config | YAML, no hardcoded secrets |

---

## Workflow Rules

1. **Research first** — Don't write code until requirements are clear
2. **Test before commit** — `hermes test` or relevant test suite
3. **Commit message format** — `type: short description`
4. **No secrets in repo** — Use environment variables

---

## Tool Usage Expectations

| Tool | When to Use |
|------|-------------|
| LM Studio | Local GGUF inference (Qwen3.5, Gemma 4) |
| OpenRouter | Cloud API (Qwen3.6-plus, MiniMax M2.7) |
| Whisper | Audio transcription on NUC |
| Syncthing | File sync between machines |

---

## What Goes Here vs SOUL.md

| SOUL.md | AGENTS.md |
|---------|-----------|
| Who the agent is | What the project is |
| Tone and behavior | Project structure |
| Strengths and priorities | Workflow rules |
| What to avoid | Coding conventions |

---

## Maintenance

Update this file when:
- New research areas are added
- Project structure changes
- New tools are integrated
- Workflow rules evolve

Don't update this file when:
- You want to change how an agent behaves (that's SOUL.md)
- You want to add temporary notes (use a session note)
