# Credential Pools — Multi-Key Rotation & Failover

**Purpose:** Configure V0.90's credential pool system to eliminate 401/429 dead stops across all providers
**Applies to:** V0.90+, high-uptime operations, high-frequency cron jobs, cost-sensitive setups
**Last updated:** 2026-04-15

---

## What Credential Pools Solves

**The problem:**
- You have multiple OpenRouter API keys (maybe a free tier + paid)
- One key hits a 429 rate limit — your cron job fails silently
- Or worse: a key gets rate-limited mid-session, you lose context
- For production tasks: this is unacceptable

**What V0.90's Credential Pools does:**
- Define a pool of API keys per provider
- Hermes automatically rotates to the next key on 401/429
- No session interruption, no manual key swapping
- You can even assign cost weights so cheaper keys are used first

---

## The Architecture

```
Credential Pool (OpenRouter)
├── Key 1: sk-or-... (free tier, weight: 1) — label: "primary-free"
├── Key 2: sk-or-... (paid, weight: 5)         — label: "paid-1"
└── Key 3: sk-or-... (paid, weight: 5)         — label: "paid-2"

Automatic failover: on 401/429 → rotate to next key → retry
```

---

## Configuring a Credential Pool

In `config.yaml`:

```yaml
providers:
  openrouter:
    credential_pool:
      enabled: true
      keys:
        - key: "${OPENROUTER_API_KEY}"
          weight: 10
          label: "primary-free"
        - key: "${OPENROUTER_API_KEY_2}"
          weight: 9
          label: "secondary-free"
      failback_to_first: true   # After all paid keys exhausted, retry free tier
      retry_on:
        - 401
        - 429
        - 503
```

In your `.env`:

```bash
# Primary and backup free-tier keys
OPENROUTER_API_KEY=sk-or-...
OPENROUTER_API_KEY_2=sk-or-...
```

---

## How Key Rotation Works

**Request 1-100:** Key 1 (primary-free, weight 10)
**Request 101+:** Key 1 hits 429 → rotate to Key 2 (secondary-free)
**If Key 2 also exhausted:** Wait, failback to Key 1 after cooldown

**Weights** determine cost preference — if you want to save paid credits for emergencies:

```yaml
keys:
  - key: "${OPENROUTER_API_KEY_FREE}"
    weight: 10       # Use free key first (high weight = preferred)
  - key: "${OPENROUTER_API_KEY_PAID}"
    weight: 1         # Use paid key only as fallback
```

---

## Per-Provider Pools

You can configure pools per provider independently:

```yaml
providers:
  openrouter:
    credential_pool:
      enabled: true
      keys:
        - key: "${OPENROUTER_KEY_1}"
        - key: "${OPENROUTER_KEY_2}"

  anthropic:
    credential_pool:
      enabled: true
      keys:
        - key: "${ANTHROPIC_KEY_1}"
        - key: "${ANTHROPIC_KEY_2}"

  google:
    # Google AI Studio has a separate free tier — pool it too
    credential_pool:
      enabled: true
      keys:
        - key: "${GOOGLE_AI_API_KEY}"
```

---

## Silent Failover Mode

For background cron jobs where you don't want notifications every time a key rotates:

```yaml
providers:
  openrouter:
    credential_pool:
      enabled: true
      silent_rotation: true   # Don't ping on key rotation, just log it
      keys:
        - key: "${OPENROUTER_KEY_1}"
        - key: "${OPENROUTER_KEY_2}"
```

The rotation is logged but doesn't fire a user notification.

---

## Health Check Between Rotations

On key rotation, Hermes can verify the new key is functional before using it:

```yaml
providers:
  openrouter:
    credential_pool:
      enabled: true
      health_check: true      # Probe new key before using it
      health_check_endpoint: "/v1/models"
      keys:
        - key: "${OPENROUTER_KEY_1}"
        - key: "${OPENROUTER_KEY_2}"
```

---

## Monitoring Your Pool Status

Check which keys are active and how much quota remains:

```bash
hermes provider status openrouter
```

Output:
```
OpenRouter Credential Pool
├── Active Key: paid-2 (sk-or-v1-...f3a2)
├── Keys Rotated: 12 times (last: 2026-04-14 08:51)
├── Quota Used Today:
│   ├── primary-free: 100/100 (exhausted)
│   ├── paid-1: 340/1000
│   └── paid-2: 127/1000
└── Pool Health: OK
```

---

## Cron-Specific Pool Usage

You can pin a cron job to a specific key or pool:

```bash
hermes cron create \
  --prompt "Run the SideHustle alpha scan..." \
  --schedule "0 3 * * *" \
  --deliver "discord" \
  --provider "openrouter" \
  --credential_pool "high-volume"  # Named pool for this job type
```

This isolates high-frequency cron jobs to their own pool so they don't burn your premium keys.

---

## Quick Setup — Your Current Free-Tier-First Stack

Since you're cost-conscious and running on free models primarily:

```bash
# Add a second free-tier key as backup
hermes config add-key openrouter OPENROUTER_API_KEY_2 sk-or-v1-...
```

Then enable the pool with your free keys prioritized:

```yaml
providers:
  openrouter:
    credential_pool:
      enabled: true
      keys:
        - key: "${OPENROUTER_API_KEY}"
          weight: 10
          label: "primary-free"
        - key: "${OPENROUTER_API_KEY_2}"
          weight: 9
          label: "secondary-free"
      failback_to_first: true
      retry_on:
        - 401
        - 429
```

This gives you **two free-tier keys** rotating automatically. No 429 dead stops on the daily driver.

---

## What You Still Handle Manually

- API key quota exhaustion (daily limits reset at midnight UTC)
- Invalid keys (401 on all keys = pool exhausted — manual intervention needed)
- Provider-level billing issues

---

## Integration with Your 3-Layer Failover

| Layer | What It Handles | Credential Pools Role |
|-------|-----------------|----------------------|
| Layer 1 | Key-level failures | ✅ Primary scope (401/429 rotation) |
| Layer 2 | Provider-level failures | ❌ Not covered — use primary fallback model |
| Layer 3 | Side-task failures | ❌ Each task type has its own routing (auto or explicit) |

**Key insight:** Credential pools are Layer 1 only. For provider outages, rely on your `fallback_model` configuration. For auxiliary tasks (vision, compression), configure explicit routing.

---

**Best Practice:** Combine credential pools (Layer 1) with primary fallback (Layer 2) and explicit auxiliary routing (Layer 3) for complete coverage across all failure modes.