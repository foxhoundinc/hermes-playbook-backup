# Workflow: X Bookmark Summarizer

**Source:** 0xJeff (2026-04-14) — 3 weeks operational experience
**Purpose:** Turn your X/Twitter bookmarks into actionable next steps every morning
**Time to run:** ~3-5 minutes automation via cron

---

## What It Does

1. Ingests your X bookmarks (from a browser export or via X API)
2. Groups by topic/thread
3. Summarizes each cluster
4. Translates into concrete "what to do / next steps"
5. Delivered to Discord or Telegram

## Why It Works

Most people bookmark 20-50 things and never look at them again. This workflow makes them actionable — Hermes reads everything and distills it into decisions you can act on.

## Cron Setup

```bash
# Fire at 7AM — after the macro brief, before your day starts
hermes cron create \
  --name "Bookmark Morning Digest" \
  --schedule "0 7 * * *" \
  --model "qwen/qwen3.6-plus" \
  --prompt "You are a research analyst. Each morning:
1. Read my X bookmarks from the bookmark file (~/.hermes/data/x-bookmarks.html)
2. Group by topic: threads, articles, tools, opportunities, random finds
3. For each group: one-sentence summary of the most important thing
4. Then: what should I DO about it? Next concrete step.
5. Format:
   THREAD: [topic] — one line
   INSIGHT: [what I learned]
   NEXT STEP: [one specific action]
6. Max 10 bookmarks. Prioritize by potential impact.
Deliver to #general on Discord."
```

## Input Data

Your X bookmarks need to be accessible. Options:
- **Browser export:** Export bookmarks as HTML, save to `~/.hermes/data/x-bookmarks.html`
- **X API:** If you have X API access, fetch bookmarks via MCP
- **Manual:** Drop a `bookmarks.md` file with links before the cron fires

## Sample Output

```
BOOKMARK DIGEST — April 14, 2026

[AI TOOLS]
🔗 "New Manus equivalent" — AI agent that executes tasks
INSIGHT: Competitor to Hermes emerging. They do browser-native agents well.
NEXT STEP: Test their free tier. Compare task completion rate vs Hermes.

[DEFI]
🔗 "New Aave lending pool" — 12% yield on USDC
INSIGHT: Fresh pool with good APY, audit pending
NEXT STEP: Wait for Trail of Bits audit before considering

[LEARNING]
🔗 "Tim Ferriss: decision journaling" — framework for high-stakes decisions
INSIGHT: Write down the decision, write down the worst outcome, write down the best outcome
NEXT STEP: Use this before next major decision (Bittensor subnet launch)

3 bookmarks actioned. 7 archived (low signal this week).
```

---

## Feedback Loop

- "Too generic" → Hermes clusters better
- "Ignored DeFi" → Hermes adds DeFi check to prompt
- "Bookmarks were stale" → Hermes adds date filter

---

*Last updated: 2026-04-14*
