# Workflow: Worldview Builder

**Source:** 0xJeff (2026-04-14) — 3 weeks operational experience
**Purpose:** Learn from smart people, synthesize into a personal worldview that evolves over time
**Time to run:** ~10 minutes, twice a week (not daily — signal-to-noise matters)

---

## What It Does

1. Monitors key feeds: X follows, newsletters, podcast highlights
2. Tracks ideas, frameworks, predictions from people you respect
3. Compares across sources — where do they agree? Disagree?
4. Builds a personal "worldview map" — what you believe, why, and what evidence supports it
5. Updates the worldview map based on new evidence

## Why It Works

Most people absorb information passively. This workflow makes learning active — Hermes tracks what you've absorbed, identifies contradictions between sources, and forces you to form a view rather than just collecting links.

## Cron Setup

```bash
# Fire Mon + Thu at 8AM
hermes cron create \
  --name "Worldview Synthesis" \
  --schedule "0 8 * * 1,4" \
  --model "minimax/minimax-m2.7" \
  --prompt "You are a thinking partner. Twice a week:
1. Read my feed inputs from (~/.hermes/data/feed-inputs/) — includes X highlights, newsletter excerpts, podcast notes
2. Identify: 3 ideas from this week's inputs worth noting
3. For each idea: who said it, what's the evidence, does it contradict anything I already believe?
4. Cross-reference with my worldview map (~/.hermes/data/worldview-map.md)
5. Update the worldview map if:
   - New evidence strengthens an existing belief
   - Contradicting evidence suggests revising a belief
   - New belief that fits no existing category
6. Output format:
   [NEW IDEAS THIS WEEK]
   1. [Idea] — [Source] — [My take]
   2. ...
   [BELIEF UPDATES]
   → [Old belief] → [Updated belief] (reason: ...)
   [NEW QUESTIONS]
   What I'm still uncertain about:
   1. ...
7. Save to ~/.hermes/data/worldview-map.md (append mode)
8. Deliver summary to Discord."
```

## Required Data

Create `~/.hermes/data/feed-inputs/` and populate with:
- `x-highlights.md` — weekly X/Twitter excerpts from people you follow
- `newsletter-notes.md` — key ideas from newsletters
- `podcast-notes.md` — Tim Ferriss, Huberman, JRE highlights

Or point Hermes at an RSS feed MCP for automatic ingestion.

## Worldview Map Structure

Create `~/.hermes/data/worldview-map.md`:

```markdown
# My Worldview Map — Updated 2026-04-14

## Core Beliefs

### AI + Automation
- AI agents will be mass-adopted in 18-36 months (confidence: 75%)
- Personal AI assistants replace browsers before they replace jobs
- Evidence: Hermes trajectory, OpenAI operator, Anthropic agents
- Counter-evidence: trust issues, setup complexity

### Investments
- [BELIEF] [CONFIDENCE %] [EVIDENCE] [STATUS]

### Health / Performance
- [BELIEF] [CONFIDENCE %] [EVIDENCE] [STATUS]

## Ideas I'm Tracking

| Idea | Source | Date added | Worth revisiting |
|------|--------|------------|-----------------|

## Things I Was Wrong About

| Belief | Why I was wrong | Date corrected |
|--------|-----------------|---------------|
```

## Sample Output

```
WORLDVIEW UPDATE — April 14, 2026

[NEW IDEAS]
1. "Regret minimaxing" — Tim Ferriss framework for decisions
   Source: JRE #2123
   My take: Powerful for one-time high-stakes decisions. Needs a process.

2. "BTC as geopolitical reserve asset" — increasingly mainstream
   Source: Multiple X follows
   My take: Real but crowded trade. Looking for asymmetric bet elsewhere.

[BELIEF UPDATE]
→ OLD: "AI agents won't be ready for 3+ years"
→ NEW: "AI agents viable in 18 months for personal use"
Reason: Hermes trajectory, OpenAI operator launch, Anthropic agents

[NEW QUESTIONS]
- What's the second-order effect when most people have AI agents?
- How does Bittensor fit in a world where AI is commoditized?
```

---

## Feedback Loop

- "Worldview is too abstract" → Hermes anchors more to specific sources
- "Need to track decisions + outcomes" → Hermes adds a decision log
- "Ideas get stale" → Hermes auto-archives beliefs older than 90 days

---

*Last updated: 2026-04-14*
