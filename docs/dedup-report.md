# Dedup Report — What We Had, What We Got

**Date:** 2026-04-12

---

## What You Already Had

From Honcho memory + session history:
- Model chain: Qwen → MiniMax → Claude (already optimal)
- Vision routing: MiniMax (already correct)
- Cron jobs: SideHustle (3AM), AILearn (6AM), Scanner (7AM)
- 41 Wondelai skills retained
- Syncthing bridge NUC ↔ Win11
- hermes-hudui on NUC
- Dashboard: hermes-ecosystem.vercel.app
- Hermes V0.80 (upgraded from V0.40 over 3 weeks)

---

## What the Orange Book Confirmed

| Topic | Your Setup | Orange Book | Status |
|-------|-----------|-------------|--------|
| Model chain | Qwen → MiniMax → Claude | Same recommended | ✅ Already optimal |
| Vision routing | MiniMax | Qwen has no vision, route to capable model | ✅ Already correct |
| Cron resilience | Fallback chain | Multi-tier fallback recommended | ✅ Already done |
| Toolsets | Enabled per need | Thin toolsets, enable on demand | ✅ Already aligned |
| Gateway | Discord + Telegram | 14 platforms, all share brain | ✅ Already correct |
| Memory layers | 3-layer (session/persistent/skills) | Same architecture | ✅ Already aligned |
| Honcho | Installed | 12-layer dialectical modeling | ✅ Already installed |

**Conclusion:** Your instincts from 3 weeks of testing matched the documented best practices. No major changes needed.

---

## What gbrain Adds on Top

gbrain is Hermes taken to production extreme (14,700+ files, 40+ skills, 20+ crons). It adds:

### 1. Knowledge Base Schema (biggest new idea)
**What:** MECE directory structure with typed cross-references
**Overlap:** None — this is new for your Enhanced Mind work
**Value for Enhanced Mind:** The "compiled truth above, timeline below" pattern maps directly onto your transcript processing workflow

### 2. Operational Disciplines
**What:** Signal detection, deterministic collectors, brain-first lookup, dream cycle
**Overlap:** Partially overlaps with your cron monitoring approach
**New for you:** The "code decides if there's work, LLM handles what to do" pattern — reduces unnecessary API calls

### 3. Skill Development Cycle
**What:** 5-step: Concept → Prototype → Evaluate → Codify → Cron
**Overlap:** General alignment with how you evaluate skills
**New:** A formal framework for building skills from recurring patterns

### 4. Sub-Agent Model Routing
**What:** Which model for which task, signal detector pattern, cost optimization
**Overlap:** You've started doing this (vision → MiniMax, text → Qwen)
**New:** A formal decision tree for model routing by task type

---

## What's NOT Worth Doubling Up On

1. **agentskills.io interoperability** — skills port between Claude Code, Hermes, Cursor, etc. Your 41 Wondelai skills are fine. Don't rebuild them.

2. **OpenClaw comparison** — you use Hermes. Don't need deep OpenClaw docs unless you're comparing for a decision.

3. **Old version docs** — you're on V0.80. Old changelogs from V0.40-V0.60 are mostly resolved bugs.

4. **Detailed Docker/VPS install guides** — you already have a working install. Skip unless migrating.

---

## Priority Reads from Downloaded Docs

If you read nothing else, read these sections:

1. **gbrain schema** — `GBRAIN_RECOMMENDED_SCHEMA.md` lines 1-200
   The MECE directory + compiled truth pattern is the single most valuable new concept

2. **Orange book** — §03 The Learning Loop, §04 Three-Layer Memory, §05 Skill System
   This is what makes Hermes different from everything else

3. **gbrain skillpack** — `operational-disciplines.md` (from the skillpack index)
   Signal detection, brain-first, cron schedule philosophy

---

## Next Step Recommendations

| Priority | Action | Why |
|----------|--------|-----|
| 🔴 High | Explore what's in your current `~/.hermes/skills/` | 3 weeks of auto-created skills you haven't seen |
| 🔴 High | Map gbrain MECE schema onto Enhanced Mind folder structure | Could unify your research library |
| 🟡 Medium | Read `operational-disciplines.md` from gbrain | The "deterministic collector" pattern saves API costs |
| 🟡 Medium | Check Carnice-MoE GGUF on Win11 | Local inference = zero API cost for certain tasks |
| 🟢 Low | Full hermes-hud review | joeynyc's HUD project — see if it adds to your dashboard |
