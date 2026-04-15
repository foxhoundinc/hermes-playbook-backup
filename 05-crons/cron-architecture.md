# Cron Job Architecture

**Source:** gbrain skillpack + Hermes Agent orange book

---

## Cron Best Practices (from 20+ production crons)

### Scheduling Principles
- **Stagger jobs** — don't bunch everything at the top of the hour
- **Quiet hours** — respect your sleep schedule. Hold non-critical jobs.
- **Timezone-aware** — crons fire on NUC time (AEST?). Make sure that's intentional.
- **Dream cycle** — gbrain runs heavy brain maintenance during off hours

### Your Current Crons
| Job | Time | Model | Status |
|-----|------|-------|--------|
| SideHustle | 3AM | Qwen3.6-plus | ? |
| AILearn | 6AM | Qwen3.6-plus | ? |
| Scanner | 7AM | Qwen3.6-plus / MiniMax | ? |

**Question:** Are these firing from the NUC or Win11? With 8GB NUC, they should fire from Win11.

---

## Resilience Patterns

### Multi-tier Fallback (already in place)
```yaml
# In cron prompt or config
model: qwen/qwen3.6-plus
fallback: minimax/minimax-m2.7
final_fallback: anthropic/claude-sonnet-4
```

### Per-Job Model Override
```bash
# Vision task — route to MiniMax
/cron create --name="Morning Vision Scan" \
  --model="minimax/minimax-m2.7" \
  --prompt="Scan screenshots and report..."

# Text research — Qwen
/cron create --name="Daily Research" \
  --model="qwen/qwen3.6-plus" \
  --prompt="Research latest AI developments..."
```

---

## What to Monitor

| Check | How |
|-------|-----|
| Cron fired? | Discord/Telegram confirmation message |
| Output delivered? | Check target channel |
| Rate limited? | Check cron output for 429 errors |
| Model dead? | Fallback kicked in? |

Use `cron-job-diagnostics` skill when something goes wrong.

---

## Signal Detector Pattern (gbrain)

Before firing any cron, the agent should:
1. Check if there's actually new signal (new emails, new github activity, etc.)
2. Skip if nothing new — don't waste API calls
3. Only run if there's fresh data to process

This is the "deterministic collector" pattern: code decides if there's work, LLM handles the judgment of what to do with it.

---

## Operational Discipline

From gbrain's `operational-disciplines.md`:
1. **Signal detection** — detect new data before processing
2. **Brain-first lookup** — check existing knowledge before calling APIs
3. **Sync-after-write** — after any write op, sync the index
4. **Heartbeat** — regular lightweight ping to confirm system alive
5. **Dream cycle** — heavy processing during user's off hours

---

## gbrain Reference Cron Schedule

gbrain runs 20+ recurring jobs including:
- Meeting sync ( Circleback → brain)
- Email triage (Gmail → entity pages)
- X/Twitter monitoring
- Calendar-to-brain
- Brain health checks
- Skill linting

The exact schedule is in:
`~/HQ/HERMESHQ/hermes-playbook/06-tools/gbrain-docs/GBRAIN_SKILLPACK.md` → `guides/cron-schedule.md`
