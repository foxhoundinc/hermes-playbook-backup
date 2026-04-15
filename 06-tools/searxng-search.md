# SearXNG — Private Web Search for Local LLMs

**Purpose:** Install SearXNG on your NUC as a self-hosted metasearch engine, giving local LLMs live web access without API costs
**Runs on:** NUC (Arch Linux) — Docker recommended
**Source:** https://github.com/searxng/searxng + https://docs.searxng.org
**Last updated:** 2026-04-14

---

## Why SearXNG for Local LLMs

Your local models (Qwen via LM Studio) are great at reasoning but have no live web data. They can't answer "what's happening in crypto today" or verify current information.

SearXNG fixes that — privacy-respecting, self-hosted metasearch that:
- Aggregates from 80+ search engines (Google, Bing, DuckDuckGo, Wikipedia, etc.)
- Runs entirely on your hardware — no data leaves your network
- Is free to run — no API costs per search
- Exposes a clean JSON API — can be called by Hermes or directly by a local LLM

**The stack:**
```
LM Studio (Qwen 3.5-35B on RTX 5070 Ti)
         ↓  [via Hermes tool call]
SearXNG (self-hosted on NUC, port 8080)
         ↓
Aggregated live web search results
         ↓
Context injected into local LLM session
```

---

## The Intel (from @TheAhmadOsman on X)

> "Using local LLMs? Make sure to setup web search for them. Tell your favorite agent to setup SearXNG for you. Give that to your local LLMs. Watch them become way more intelligent and efficient."

The core insight: local LLMs without search are effectively blind to the present. SearXNG makes them current.

---

## Install on NUC — Docker Compose (Recommended)

SearXNG runs cleanly in Docker on Arch Linux. This is the fastest path.

### 1. Check Docker on NUC

```bash
# If Docker isn't installed yet (Arch Linux)
sudo pacman -S docker
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
# Log out and back in for group membership to take effect

# Verify
docker --version
docker compose version
```

### 2. Create the SearXNG Environment

```bash
# On your NUC
mkdir -p ~/HQ/HERMESHQ/searxng
cd ~/HQ/HERMESHQ/searxng

# Fetch the official Docker Compose template
curl -fsSL \
  -O https://raw.githubusercontent.com/searxng/searxng/master/container/docker-compose.yml \
  -O https://raw.githubusercontent.com/searxng/searxng/master/container/.env.example

cp .env.example .env
```

### 3. Configure SearXNG

Edit `.env` to set `SEARXNG_BASE_URL` to your NUC's local IP:

```bash
nano .env
```

Key variables:
```bash
SEARXNG_BASE_URL=http://192.168.172.238:8080   # Your NUC's LAN IP
SEARXNG_SECRET_KEY=generate-a-random-string    # python3 -c "import secrets; print(secrets.token_hex(32))"
SEARXNG_INSTANCE_NAME=hermes-nuc-searxng
```

Create the config directory:
```bash
mkdir -p ./core-config
```

Edit `./core-config/settings.yml` to enable JSON output and adjust for LLM use:
```bash
nano ./core-config/settings.yml
```

Key settings for an LLM-facing instance:
```yaml
general:
  debug: false
  instance_name: hermes-nuc-searxng
  privac level: metasearch  # Don't log searches

search:
  formats:
    - json          # Enable JSON output for API calls
    - html
  autocomplete: "google"  # Or leave empty, use an engine's autocomplete

server:
  bind_address: "0.0.0.0"  # Listen on all interfaces so Win11 can reach it
  port: 8080
  secret_key: "${SEARXNG_SECRET_KEY}"
```

### 4. Start SearXNG

```bash
docker compose up -d
```

Verify:
```bash
docker compose ps
# Expected: searxng-core Up, searxng-valkey Up

# Test from NUC
curl "http://localhost:8080/search?q=bitcoin+price&format=json" | head -100
```

### 5. Allow Win11 to Reach SearXNG

By default, Docker containers aren't reachable from the LAN. Fix this:

```bash
# Update docker-compose.yml to expose on all interfaces
# Change the port binding from "127.0.0.1:8080:8080" to "0.0.0.0:8080:8080"

# Or add a firewall rule on NUC (Arch Linux)
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Or for a quick test (less secure)
sudo iptables -I INPUT -p tcp --dport 8080 -j ACCEPT
```

Then test from Win11 browser: `http://192.168.172.238:8080`

---

## Connect SearXNG to Hermes

### Option A — Hermes web_search Tool (Easiest)

Point Hermes's web search at your SearXNG instance by setting an environment variable or config:

```bash
# In your .env on Win11 (WSL2 CachyOS)
export SEARXNG_URL=http://192.168.172.238:8080

# Or in config.yaml
search:
  provider: searxng
  base_url: http://192.168.172.238:8080
```

When Hermes runs a web search, it calls your private SearXNG instead of an external API.

### Option B — Direct LLM API Integration

If your local LLM tool can call custom endpoints, point it at:

```
http://192.168.172.238:8080/search?q={query}&format=json
```

### Test the Integration

```bash
# From Win11 (WSL2)
curl "http://192.168.172.238:8080/search?q=ethereum+gas+price&format=json" 2>/dev/null | python3 -c "
import json, sys
data = json.load(sys.stdin)
results = data.get('results', [])[:3]
for r in results:
    print(r['title'])
    print(r['url'])
    print()
"
```

Expected: Top 3 search results for Ethereum gas price.

---

## SearXNG API Reference

**Search endpoint:**
```
GET http://<nuc-ip>:8080/search?q=<query>&format=json
```

**Autocomplete endpoint:**
```
GET http://<nuc-ip>:8080/autocomple?q=<prefix>
```

**JSON response structure:**
```json
{
  "query": "...",
  "results": [
    {
      "title": "...",
      "url": "...",
      "content": "...",
      "engines": ["google", "bing"]
    }
  ],
  "number_of_results": N,
  "infoboxes": [],
  "suggestions": []
}
```

---

## Useful Config Tweaks for LLM Use

In `./core-config/settings.yml`:

```yaml
# Disable result caching — always fresh for LLM context
search:
  cache_length: 0          # No cache for research tasks

# Enable safe search by default
search:
  safe_search: 1            # 0=none, 1=moderate, 2=strict

# Limit results to reduce noise
search:
  max_value: 10             # Return top 10 results only

# Enable HTML stripping for cleaner LLM context
search:
  result_template: false
```

---

## Updating SearXNG

```bash
cd ~/HQ/HERMESHQ/searxng
docker compose down
docker compose pull
docker compose up -d
```

---

## Troubleshooting

**"Connection refused" from Win11**
→ SearXNG is binding to 127.0.0.1 instead of 0.0.0.0
→ Edit `docker-compose.yml`, change `127.0.0.1:8080:8080` to `0.0.0.0:8080:8080`

**Empty search results**
→ Check: `docker compose logs core` for errors
→ Verify network: `curl http://localhost:8080` from NUC itself

**Rate limiting from upstream engines**
→ SearXNG throttles requests automatically
→ For heavy LLM use, consider enabling engines with higher rate limits

---

## Quick Reference

| What | Command/Value |
|------|--------------|
| Start | `docker compose up -d` |
| Stop | `docker compose down` |
| Logs | `docker compose logs -f core` |
| Shell in container | `docker compose exec -it core /bin/sh` |
| Web UI | `http://192.168.172.238:8080` |
| JSON API | `http://192.168.172.238:8080/search?q=<query>&format=json` |
| Config file | `./core-config/settings.yml` |
| Update | `docker compose pull && docker compose up -d` |
