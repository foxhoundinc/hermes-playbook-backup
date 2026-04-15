# Resilience Stack — Three Layers of Failover

**Purpose:** Document V0.90's layered failover architecture so you configure it deliberately, not accidentally
**Applies to:** V0.90+, any serious operational setup

---

## The Mental Model

Most users think of fallback as a toggle. V0.90 treats it as a **three-layer system**, where each layer solves a different class of failure:

```
Layer 1: Credential Pools     → Key-level failures (one bad key, same provider)
Layer 2: Primary Fallback     → Provider-level failures (whole provider path down)
Layer 3: Auxiliary Routing    → Side-task failures (vision, compression, web_extract)
```

**The critical insight:** these layers don't overlap. Credential pools won't save you when OpenRouter itself is degraded. Primary fallback won't help your compression task. Each layer is purpose-built.

---

## Layer 1 — Credential Pools

**What it solves:** One key on a provider is bad (exhausted, revoked, limited). Same provider path.

**What it doesn't solve:** Provider-level outages, path degradation, or any failure above the credential level.

**Already documented in:** `06-tools/credential-pools.md`

**Your current setup** — two free-tier OpenRouter keys rotating automatically:

```yaml
providers:
  openrouter:
    credential_pool:
      enabled: true
      keys:
        - key: "${OPENROUTER_API_KEY}"
          weight: 10
        - key: "${OPENROUTER_API_KEY_2}"
          weight: 9
      failback_to_first: true
```

---

## Layer 2 — Primary Model Fallback

**What it solves:** The entire primary provider path is down (429 rate limits exhausted, 503 degradation, 401 auth failure, full provider outage).

**What it doesn't solve:** Side tasks (vision, compression), cron jobs, subagent delegation.

### The One-Shot Design Constraint

Primary fallback is **one-shot per session.** When the primary fails:
1. Hermes swaps to the fallback provider:model
2. Continues the session with full context preserved
3. If the fallback also fails → normal error handling. No third bounce.

**Why this matters:** Endless failover chains create unpredictable behavior and are harder to debug. One controlled handoff is better than cascading failures.

### How to Configure

In `config.yaml`:

```yaml
model:
  provider: openrouter
  default: qwen/qwen3.6-plus

fallback_model:
  provider: openrouter
  model: minimax/minimax-m2.7
```

**Both fields required.** If `provider` or `model` is missing, fallback is disabled silently.

### Trigger Conditions

Fallback activates after retries are exhausted on:
- HTTP 429 (rate limited)
- HTTP 500, 502, 503 (server errors)
- HTTP 401, 403 (auth failures)
- HTTP 404 (model not found)
- Repeated malformed or empty API responses

### What Gets Preserved on Fallback

The session state survives the swap:
- Full conversation history
- Context (USER.md, memories, Honcho state)
- Tool flow and tool state
- Place in the current task

What resets: retry counters and the fallback itself (it won't fire again in that session).

### Choosing a Fallback Pair

Optimize for one of three things:

| Strategy | Best for | Example |
|---------|---------|---------|
| **Similar capability** | Writing, reasoning, high-context work where style continuity matters | Anthropic Sonnet 4 → OpenRouter Claude Sonnet 4 |
| **Infrastructure diversity** | Avoiding single-provider failure domains | Anthropic native → OpenRouter (different infrastructure) |
| **Cost containment** | Long-running background workflows where "good enough" is better than stopping | Primary paid → fallback free tier |

**Your current stack — recommended fallback:**
```yaml
model:
  provider: openrouter
  default: qwen/qwen3.6-plus

fallback_model:
  provider: openrouter
  model: minimax/minimax-m2.7
```

Both on OpenRouter (same infrastructure), both free-tier, different model families for diversity.

**Phase 2 upgrade** (when you have budget):
```yaml
fallback_model:
  provider: openrouter
  model: anthropic/claude-sonnet-4-6
```

---

## Layer 3 — Auxiliary Task Routing

**What it solves:** Side tasks like vision, compression, web extraction each need their own provider path. If your main model provider is down, your vision requests shouldn't be blocked by the same outage.

**What it doesn't solve:** Primary session fallback. Auxiliary routing is completely separate.

### What Uses Auxiliary Routing

These tasks each route independently:
- `vision` — image analysis, screenshot understanding
- `web_extract` — content extraction from URLs
- `compression` — context summarization for long sessions
- `session_search` — searching conversation history
- `skills_hub` — skill lookup and installation
- `mcp` — MCP tool calls
- `flush_memories` — memory persistence operations

### The `provider: auto` Pattern

`auto` tells Hermes to try providers in priority order until one works. This is the default and works for most setups:

```yaml
auxiliary:
  vision:
    provider: "auto"
  compression:
    provider: "auto"
  web_extract:
    provider: "auto"
```

### Explicit Auxiliary Configuration

When you need a specific provider for a specific task:

```yaml
auxiliary:
  vision:
    provider: "openrouter"
    model: "google/gemma-4-31b"    # Vision + reasoning on Gemma 4

  compression:
    provider: "openrouter"
    model: "google/gemini-3-flash-preview"  # Fast, cheap summarization

  web_extract:
    provider: "auto"    # Let Hermes find what works
```

### Graceful Degradation — The Compression Example

When context gets large, Hermes may compress via summarization. If no compression provider is available:

- **Good:** Hermes drops middle conversation turns without a summary
- **Bad:** Session crashes

This is intentional. The system degrades to "less context" rather than "complete failure." That's the right trade-off.

---

## What Does NOT Inherit Primary Fallback

This is the gap that trips most users up:

| System | Primary Fallback? | What to do instead |
|--------|------------------|-------------------|
| **Subagent delegation** | No | Assign provider:model override per subagent explicitly |
| **Cron jobs** | No | Configure provider at execution time or via per-job model override |
| **Auxiliary tasks** | No | Each has its own routing (auto or explicit) |

If you care about reliability in subagents and cron jobs, configure those paths explicitly. They're not covered by `fallback_model`.

---

## The Full Config — Your Production Stack

```yaml
# ~/.hermes/config.yaml

# Layer 1: Credential Pools
providers:
  openrouter:
    credential_pool:
      enabled: true
      keys:
        - key: "${OPENROUTER_API_KEY}"
          weight: 10
        - key: "${OPENROUTER_API_KEY_2}"
          weight: 9
      failback_to_first: true
      retry_on:
        - 401
        - 429
        - 503

# Layer 2: Primary Model Fallback
model:
  provider: openrouter
  default: qwen/qwen3.6-plus

fallback_model:
  provider: openrouter
  model: minimax/minimax-m2.7

# Layer 3: Auxiliary Task Routing
auxiliary:
  vision:
    provider: "openrouter"
    model: "google/gemma-4-31b"
  compression:
    provider: "auto"
  web_extract:
    provider: "auto"
  session_search:
    provider: "auto"
  skills_hub:
    provider: "auto"
  mcp:
    provider: "auto"
```

---

## The Reliability Upgrade Path

| Phase | What's configured | Failover capability |
|-------|------------------|-------------------|
| **Now** | Layer 1 only (credential pools) | Key rotation only |
| **After this guide** | Layer 1 + Layer 2 | Key rotation + session failover |
| **Phase 2** | All 3 layers + Signal sentinel | Full stack + proactive alerts |
| **Phase 3** | Local LM Studio as fallback | Cloud-independent resilience |

---

## Quick Diagnostic

If something breaks, which layer failed?

```
Did a request fail with 401/429 on same provider?    → Layer 1: check credential pool
Did the main session stop mid-task with provider error? → Layer 2: check fallback_model config
Did image analysis fail but chat still works?        → Layer 3: check auxiliary.vision config
Did a cron job silently fail?                        → Not covered by fallback — configure cron explicitly
```

---

## Notes

- Subagent delegation, cron jobs, and auxiliary tasks do **not** inherit the primary `fallback_model`
- Configure those paths explicitly if you need reliability there
- Auxiliary routing defaults to `provider: auto` — use explicit configuration only when you need a specific provider