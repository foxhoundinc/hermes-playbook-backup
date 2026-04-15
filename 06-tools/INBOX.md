# 06-tools — Tool-Specific Optimizations

> **INBOX:** Unsorted links, PDFs, and notes go here first. Sort into sub-docs after review.

## ✅ Processed — Downloaded & Indexed
- (2026-04-12) **gbrain** — garrytan's opinionated Hermes brain system
  - Full repo cloned to: `hermes-playbook/06-tools/gbrain-docs/`
  - Key files:
    - `GBRAIN_RECOMMENDED_SCHEMA.md` — MECE knowledge base schema (Karpathy wiki pattern extended)
    - `GBRAIN_SKILLPACK.md` — 40+ skills, 20+ crons, 14,700+ brain files of operational patterns
    - `GBRAIN_V0.md` — Version-specific documentation
    - `GBRAIN_VERIFY.md` — Installation verification runbook
  - Schema pattern: Compiled truth (above line) + Timeline (below line) + typed backlinks
  - Architecture: Raw sources → brain (markdown wiki) → schema (conventions)
  - Integration recipes: voice, email, X, calendar, GitHub MCP
- (2026-04-14) **backup-guide.md** — V0.90 backup system documentation
  - Full file: `hermes-playbook/06-tools/backup-guide.md`
  - Covers: `--quick` snapshots (new), full backup zip with `-o` flag, restore commands, schedule, disaster recovery
  - Verified commands: `hermes backup -o ~/hermes-backup-full-$(date +%Y%m%d_%H%M%S).zip`
- (2026-04-14) **sentinel-mode.md** — watch_patterns operational guide
  - Full file: `hermes-playbook/06-tools/sentinel-mode.md`
  - Covers: crypto arbitrage sentinel, GPU temp monitor, build failure sentinel, training session monitor
  - Signal delivery integration, pattern syntax (string + regex), cron integration
- (2026-04-14) **resilience-stack.md** — 3-layer failover architecture (from NousResearch docs)
  - Full file: `hermes-playbook/06-tools/resilience-stack.md`
  - Layer 1: Credential pools (key rotation) — already had credential-pools.md, this ties it to the full stack
  - Layer 2: Primary model fallback (fallback_model: YAML block, one-shot design)
  - Layer 3: Auxiliary task routing (vision/compression/web_extract/session_search — NEW gap filled)
  - Critical gap filled: subagent/cron do NOT inherit primary fallback — must be configured explicitly
  - Practical fallback pair for current stack: qwen3.6-plus → minimax-m2.7 (both OpenRouter free)
  - Full file: `hermes-playbook/06-tools/credential-pools.md`
  - Covers: credential pool config (YAML), weight-based routing, 401/429/503 failover, health checks
  - Per-provider pools, silent rotation mode, cron isolation, monitoring with `hermes provider status`

- (2026-04-14) **searxng-search.md** — Private web search for local LLMs
  - Source: @TheAhmadOsman (X/Twitter) + https://docs.searxng.org
  - Full file: `hermes-playbook/06-tools/searxng-search.md`
  - Runs on: NUC (Docker Compose), Arch Linux compatible
  - Setup: Docker Compose, JSON API at port 8080, exposed to Win11 LAN
  - Integration: Hermes web_search tool points to SearXNG, or direct LLM API calls
  - Search: 80+ engines aggregated, no API costs, all traffic stays local
  - Note: searxng-docker repo deprecated — use official compose template from searxng/searxxng master

## Sub-docs (to be created)
- `tool-master-list.md` — All tools indexed
- `web-research.md` — web_search / web_extract best practices
- `browser-automation.md` — Browser tool usage
- `audio-transcription.md` — Whisper workflow on NUC
- `delegate-tool.md` — Subagent orchestration
- `mcp-tools.md` — MCP server integrations

---
*Sort contents into sub-docs after reviewing*
