# 05-multi-agent — Profile Team Setup

**Source:** Neo's multi-agent architecture article
**Purpose:** Build a team of specialized Hermes profiles instead of one overloaded agent

---

## The Problem

A single agent doing everything (research, writing, coding, debugging) gets:
- Bloated context windows
- Inconsistent output quality
- Slower responses

## The Solution

Run multiple isolated Hermes profiles, each with:
- **SOUL.md** — dedicated identity
- **Own memory** — separate session history
- **Own skills** — domain-specific
- **Shared context** — AGENTS.md for project knowledge

## Your 4-Role Team

| Role | Name | Specialty |
|------|------|-----------|
| Orchestrator | Hermes (default) | Planning, delegation, synthesis, QA |
| Research | Alan | Evidence, verification, depth |
| Writing | Mira | Clarity, structure, audience |
| Debugging | Turing | Implementation, tests, precision |

---

## Files in This Section

| File | Purpose |
|------|---------|
| `01-profile-setup.md` | Create alan, mira, turing profiles |
| `02-SOUL-md-templates.md` | SOUL.md for each role |
| `03-AGENTS-md.md` | Shared Enhanced Mind project context |
| `04-team-agents-md.md` | Roster + handoff rules |
| `05-auxiliary-models.md` | 8 dedicated model types per task |

---

## Quick Start

```bash
# Create profiles
hermes profile create alan
hermes profile create mira
hermes profile create turing

# Switch between profiles
hermes -p alan chat
hermes -p mira chat
hermes -p turing chat

# Default hermes profile (orchestrator)
hermes chat
```

---

*Source: Neo's multi-agent profile architecture guide (2026-04-13)*
