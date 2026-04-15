# Skill Development Cycle

**Source:** gbrain skillpack + Hermes Agent orange book

gbrain's author (garrytan) ran 14,700+ brain files, 40+ skills, 20+ cron jobs. The skill development cycle is the most valuable operational pattern from that experience.

---

## The 5-Step Cycle

```
Concept → Prototype → Evaluate → Codify → Cron
    ↑__________________________________|
```

### Step 1 — Concept
Identify a recurring task pattern. Things you do more than once a week. Document:
- What triggers this
- What the desired output is
- What you've tried before

### Step 2 — Prototype
Let Hermes do it manually the first 2-3 times. Don't interfere. Watch how it solves it.

### Step 3 — Evaluate
After 3 runs, review:
- What worked consistently?
- What edge cases came up?
- What would make it faster/better?

### Step 4 — Codify
Write the skill file. Include:
- **Trigger:** When to activate (be specific, not vague)
- **Rules:** Concrete steps with constraints
- **Example:** Full input → output
- **Don'ts:** Explicit boundaries

```markdown
##-
name: github-pr-review
description: Review GitHub PRs against team standards
version: "1.0.0"
##-

# GitHub PR Review Skill
## Trigger
When user asks me to review a PR, check a pull request, or audit code changes.

## Rules
1. Fetch PR diff and list of changed files
2. Check for: test coverage, security issues, performance concerns
3. Flag: files without tests, secrets in code, large changes without explanation
4. Summarize: overall health score + key findings

## Example
Input: "Review PR #42 in the frontend repo"
Output: "Health: 72/100. Concerns: auth.js lacks tests, 3 TODOs should be tracked..."
```

### Step 5 — Cron
Automate the execution:
- Daily/weekly recurring → cron job
- Event-triggered → MCP integration
- Both → cron + agent monitoring

---

## Skill Quality Gates

| Gate | Criteria |
|------|----------|
| Trigger specificity | "When the user mentions code" = too vague |
| Example completeness | Full input → full output, not just description |
| Error handling | What to do when step 3 fails |
| Boundaries | Explicit don'ts prevent scope creep |

---

## Skill Conflicts

If two skills match the same trigger, Hermes picks the higher match score. But results may be unexpected.

**Prevention:**
- Review skills quarterly
- Check for overlap when adding new skills
- Name skills to be specific about scope

**Resolution:**
```markdown
## Trigger (refined)
When user asks to commit code AND mentions "conventional commits"
# vs
When user asks me to commit code generally
```

---

## Auto-Improvement

Hermes auto-improves skills from feedback. After each skill run:
- User corrections → skill file updated
- New patterns → skill enriched
- You can also manually edit skill files anytime

**Don't let skills rot.** Review `~/.hermes/skills/` monthly.

---

## Your Current State

From your notes (2026-04-09): 41 Wondelai skills retained, 2 proposed rejected.
Your skill folder likely already has auto-created skills from your 3 weeks of use. Worth a look:

```bash
ls ~/.hermes/skills/
cat ~/.hermes/skills/*.md | head -200  # preview what's there
```

---

## gbrain Skillpack References (downloaded)

Full guides available in:
`~/HQ/HERMESHQ/hermes-playbook/06-tools/gbrain-docs/GBRAIN_SKILLPACK.md`

Key guides referenced:
- `brain-agent-loop.md` — The read-write cycle
- `entity-detection.md` — Capture entities on every signal
- `enrichment-pipeline.md` — 7-step protocol for processing ingests
- `skill-development.md` — Full cycle documentation
- `operational-disciplines.md` — Signal detection, brain-first, sync-after-write
