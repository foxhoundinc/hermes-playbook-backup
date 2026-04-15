# Sentinel Mode — Proactive Monitoring with watch_patterns

**Purpose:** Use V0.90's `watch_patterns` to run Hermes as a live sentinel that pings you when specific events fire
**Applies to:** V0.90+, background cron processes, Web3 monitoring, DevOps
**Last updated:** 2026-04-15

---

## What watch_patterns Does

`watch_patterns` is a background process monitor built into Hermes's terminal runner. Instead of polling or waiting for a cron to fire, you attach regex or string triggers to any background process — and Hermes pings you the instant a match is found.

**Without Sentinel Mode:**
```
You → Check prices manually → Miss the spike → Miss the trade window
```

**With Sentinel Mode:**
```
Your arbitrage monitor fires "High Value" in output
→ Hermes pings you on Signal immediately
→ You act within seconds, not minutes
```

---

## The Core Pattern

`watch_patterns` is passed as a parameter to background terminal processes. The system notifies you when any of the watch patterns match in the process output.

```bash
# Basic syntax (in a cron job prompt or terminal background call)
terminal(background=true, watch_patterns=["ERROR", "gas spike", "High Value"])
```

When any string in the list appears in the process output, Hermes fires a notification.

---

## Use Case 1 — Crypto Arbitrage Sentinel

**Setup:** A cron job that monitors DEX prices or contract interactions.

In your cron job prompt, attach the sentinel:

```bash
# Cron that watches for arbitrage signals
hermes cron create \
  --prompt "Monitor Ethereum gas prices via Etherscan API. Watch for gas dropping below 20 gwei AND a matching arbitrage window on Uniswap. Alert on: 'gas below 20', 'arb opportunity found', 'High Value'." \
  --schedule "*/5 * * * *" \
  --deliver "signal" \
  --watch_patterns '["gas below 20", "arb opportunity found", "High Value"]'
```

**Signal delivery** requires the Signal gateway configured in V0.90:
- Signal bot token in `.env`: `SIGNAL_BOT_TOKEN=***`
- Recipient configured in `channel_directory.json`

---

## Use Case 2 — GPU Temp Sentinel

**Setup:** Monitor your RTX 5070 Ti temperatures during a long training run.

```bash
# Watch for GPU overheat or thermal throttling
watch_patterns='["GPU temp", "temperature", "thermal", "WARNING", "hot", ">80C"]'
```

In practice — attach to a background `nvidia-smi` loop:

```bash
hermes cron create \
  --prompt "Run: watch -n 5 nvidia-smi --query-gpu=temperature.gpu,fan.speed,power.draw --format=csv. Alert if: temp > 82C, fan > 85%, or power > 350W." \
  --schedule "*/1 * * * *" \
  --deliver "discord" \
  --watch_patterns '["8[2-9]|9[0-9]", "thermal", "throttling"]'
```

The `watch_patterns` regex catches temperature spikes before they cause instability.

---

## Use Case 3 — Build Failure Sentinel

**Setup:** Monitor a code build or test suite running in the background.

```bash
# Run pytest in background, watch for failures
hermes cron create \
  --prompt "Run: cd ~/hermes-hudui && python -m pytest tests/ -x -v. Alert on: 'FAILED', 'ERROR', 'AssertionError', 'ImportError'." \
  --schedule "0 2 * * *" \
  --deliver "telegram" \
  --watch_patterns='["FAILED", "ERROR", "AssertionError", "ImportError", "Traceback"]'
```

Delivered to Telegram as soon as any error pattern fires.

---

## Use Case 4 — Training Session Monitor

**Setup:** Watch a model training run for anomalies.

```bash
hermes cron create \
  --prompt "Tail the training log at ~/training/logs/session.log. Watch for: 'loss: nan', 'OOM', 'GPU exhausted', 'epoch complete', 'val_loss improved'." \
  --schedule "0 */1 * * *" \
  --deliver "signal" \
  --watch_patterns='["loss: nan", "OOM", "GPU exhausted", "val_loss improved"]'
```

Training crashes caught in real-time, before you lose a full epoch.

---

## How watch_patterns Integrates with Hermes

The `watch_patterns` system is part of Hermes's background process registry. Each pattern is checked against every new line of process output. On match:

1. Process output up to the match line is captured
2. Hermes composes a notification with the matched line + context
3. Notification delivered to your configured channel (Signal, Telegram, Discord)
4. Process continues running (not terminated)

**The delivery channel** is determined by your cron job's `--deliver` setting.

---

## watch_patterns + Cron Schedule

Sentinels are cron jobs that run continuously and fire only when patterns match. You don't need short intervals — use event-triggered logic inside the cron prompt itself:

```bash
# Good: Cron runs periodically, prompt does the monitoring
hermes cron create \
  --prompt "Check gas prices every cycle. Only respond via Signal when: 'gas < 20 AND arb opportunity detected'." \
  --schedule "*/10 * * * *" \
  --deliver "signal"
  --watch_patterns='["arb opportunity"]'
```

The `watch_patterns` acts as a safety net — catches any output line that matches even if the prompt logic didn't trigger.

---

## Signal Setup for Sentinel Alerts

Signal is V0.90's encrypted delivery channel for sensitive alerts (crypto, financial).

### 1. Get a Signal Bot Token

Signal bots use a bridge service (like signal-cli or a commercial bridge). The simplest setup for personal use:
- Signal-connected phone number as the bot identity
- Configure the Signal bridge in your `.env`:

```bash
SIGNAL_BRIDGE_URL=http://localhost:9080
SIGNAL_BOT_TOKEN=***
```

### 2. Configure Recipient

In `channel_directory.json`, add your Signal number:

```json
{
  "signal": "+155****4567"
}
```

### 3. Test It

```bash
# Send a test Signal
hermes send --channel signal --message "Sentinel test — GPU monitor active"
```

---

## Quick Reference

| Pattern Type | Example | Match Triggers |
|-------------|---------|---------------|
| String match | `"gas spike"` | Exact substring in output |
| Regex range | `"8[2-9]"` | Temperature 82-89 |
| Multiple | `["ERROR", "FAILED"]` | Any of the listed terms |
| Case-insensitive | `"high value"` | Matches "High Value", "HIGH VALUE" |

**Note:** Pattern matching is case-insensitive by default.

---

## Troubleshooting

**"Patterns not firing"**
- Check that `--deliver` channel is properly configured
- Verify the process output actually contains the pattern text
- Ensure `background=true` is set on the terminal call

**"Getting too many notifications"**
- Refine your regex patterns to be more specific
- Use word boundaries: `"\bERROR\b"` instead of just `"ERROR"`
- Add negative patterns in your prompt logic

**"Signal not receiving"**
- Verify `SIGNAL_BOT_TOKEN` is set in `.env`
- Check `channel_directory.json` has the correct Signal number
- Ensure the Signal bridge service is running

---

## Integration with Other V0.90 Features

- **Credential Pools:** Works independently — sentinel patterns fire regardless of key rotation
- **Primary Fallback:** If main model fails, sentinel continues to fire on whatever output is generated
- **Auxiliary Routing:** Separate from sentinel — vision/web tasks can have their own pattern monitoring
- **Honcho Memory:** Pattern matches and notifications are logged in session memory for post-analysis

---

## Best Practices

1. **Be specific with patterns** — use regex to avoid false positives
2. **Use multiple patterns** — catch different failure modes
3. **Test on dummy output** — verify patterns match before deploying to production
4. **Set appropriate `--deliver` channels** — Signal for critical alerts, Discord for warnings
5. **Log pattern matches** — use Honcho memory to review what fired and when

---

*Last updated: 2026-04-15*