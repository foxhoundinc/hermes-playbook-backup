# 07-troubleshooting — Known Issues & Fixes

> **INBOX:** Unsorted links, PDFs, and notes go here first. Sort into sub-docs after review.

## Known Issues (from past sessions)
- `rm -rf` hangs on cloud sandbox → use Python `shutil.rmtree()` instead
- Dashboard `/api/dashboard` 500 error — intermittent, resolved by restart
- Vision tool not loading — run `vision-tool-diagnostics` skill
- Hermes updates every ~5 days — check `hermes-update` skill before updating

## Windows + Hermes WSL2 Fix (0xJeff, 2026-04-14)
**Problem:** Hermes on Windows direct folder path causes problems with commands, running cron jobs, and editing configs. Works great on Mac, breaks on Windows.

**Fix:** Reinstall Hermes through WSL2 via Ubuntu — works great afterwards.
```powershell
# Install WSL2 with Ubuntu on Windows
wsl --install -d Ubuntu
# Then install Hermes inside WSL2
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/install.sh | sh
```

**Why it works:** WSL2 provides a Linux-compatible environment where Hermes paths and commands behave as designed.

## Claude Max Deprecation (0xJeff, 2026-04-14)
**Issue:** Anthropic banned agents 1-2 weeks back. Cannot use Hermes with Claude subscription credits anymore.

**Fix:** Use OpenRouter for Claude access instead of direct Claude subscription:
- Claude Sonnet 4 via OpenRouter still works (your current fallback)
- Do NOT subscribe to Claude Max — Anthropic blocks agent access to subscription credits
- Cost-effective alternative: OpenRouter + free models (Qwen3.6-plus) + Kimi/GLM for heavy tasks

## OpenRouter Model Cost Wisdom (0xJeff, 2026-04-14)
- Kimi k2.5 — great for long-running tasks, good cost/efficiency ratio
- GLM5.1 — great for long-running tasks, strong alternative
- MiniMax 2.7 — conversational flow works well
- OpenCode Go for $5/mo — saves tokens vs raw API; Kimi k2.5 and GLM5.1 work great within it

## Pending Items
- [ ] TODO: Add more known issues
- [ ] TODO: Add diagnostic workflows
- [ ] TODO: Add fix documentation

## Sub-docs (to be created)
- `known-issues.md` — All known issues + solutions
- `diagnostic-workflows.md` — How to debug systematically
- `error-codex.md` — Common errors + what they mean

---
*Sort contents into sub-docs after reviewing*
