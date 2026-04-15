# Workflow: Macro Morning Brief

**Source:** 0xJeff (2026-04-14) — 3 weeks operational experience
**Purpose:** Track macro + geopolitics + portfolio impact — delivered every morning
**Time to run:** ~5-10 minutes automation via cron

---

## What It Does

1. Monitors macro indicators (US-Iran situation, oil, key macro metrics)
2. Identifies top insights from overnight data
3. Connects insights to your portfolio — what it means for your positions
4. Delivered to Discord every morning

## Why It Works

Instead of scouring through X, news sites, checking dashboards — you open Discord and Hermes has already done the work. You provide feedback, Hermes learns and improves.

## Cron Setup

```bash
# Fire at 6AM local time — before market open
hermes cron create \
  --name "Macro Morning Brief" \
  --schedule "0 6 * * *" \
  --model "minimax/minimax-m2.7" \
  --prompt "You are a macro analyst. Each morning:
1. Check overnight macro: US-Iran situation headlines, oil price (WTI), dollar index, VIX
2. Synthesize top 3 insights
3. For each insight: what does it mean for my portfolio (crypto + traditional holdings)
4. Format as: Insight → Why it matters → Portfolio action (if any)
5. Keep to 300 words max. Be decisive — if no action needed, say so.
Deliver to #general on Discord."
```

## Skills Needed

- `web-search` — fetch overnight macro headlines
- `web-extract` — pull oil price, dollar index
- Discord posting via gateway

## Sample Output

```
MACRO MORNING BRIEF — April 14, 2026

🛢️ OIL: WTI +1.2% to $83.40
→ Iran rhetoric escalating — supply premium building
→ Your energy exposure (XLE calls): monitor stop loss at $78

📊 DXY: 104.2 (+0.3)
→ Dollar strength persisting
→ Headwinds for EM debt. Your HYB position: hold, watch credit spreads.

🌐 GEOPOLITICS: US-Iran talks stall
→ Risk-on in energy, risk-off in equities
→ Portfolio: reduce equity exposure 10%, add to cash

ACTION: No trades needed. Monitor oil at $85 for energy re-entry.
```

---

## Feedback Loop

After each brief, provide 1-line feedback:
- "Too long" → Hermes shortens
- "Ignored my crypto" → Hermes adds wallet monitoring
- "Needed more detail on oil" → Hermes deepens that section

This feedback loop is how Hermes improves — 3 weeks of this and the brief becomes highly personalized.

---

*Last updated: 2026-04-14*
