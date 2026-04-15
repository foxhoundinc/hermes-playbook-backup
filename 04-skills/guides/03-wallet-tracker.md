# Workflow: Cross-Wallet + DeFi Portfolio Tracker

**Source:** 0xJeff (2026-04-14) — 3 weeks operational experience
**Purpose:** Report all holdings across wallets and DeFi positions with latest price movements every morning
**Time to run:** ~5 minutes via cron

---

## What It Does

1. Aggregates balances across multiple wallet addresses
2. Pulls DeFi positions (lending, LP, staking) from protocols
3. Displays latest prices for all holdings
4. Calculates daily P&L
5. Flags large moves (>5% in 24h)

## Why It Works

Instead of checking Rabby, Etherscan, DeFiLlama, Coingecko separately — one command, one view, every morning. Hermes learns your positions so it can contextualize insights ("your ETH allocation is up 3% while the broader market is down 5%").

## Cron Setup

```bash
# Fire at 7:30AM — after bookmark digest
hermes cron create \
  --name "Wallet Morning Report" \
  --schedule "30 7 * * *" \
  --model "qwen/qwen3.6-plus" \
  --prompt "You are a portfolio analyst. Each morning:
1. Read my wallet addresses from (~/.hermes/data/wallets.json)
2. For each address: fetch ETH balance, any ERC-20 tokens
3. Pull DeFi positions: from DeFiLlama API for each address
4. Fetch current prices for all tokens
5. Calculate: total value in USD, 24h change ($ and %)
6. Flag: any position up/down >5% in 24h
7. Format as:
   TOTAL PORTFOLIO: $XX,XXX (+/-X.X% 24h)
   [Wallet: 0x...] — $X,XXX
     ETH: X.XX ETH @ $XXX (+X.X% 24h)
     [Token]: X.XX @ $XXX
   DEFI: $X,XXX
     Aave: X USDC deposited @ X% APY
     Uniswap: X ETH-X LP @ X% APY
   ALERTS:
     ⚠️ [Token] +X.X% — news?
     ⚠️ [Token] -X.X% — stop loss trigger?
8. Deliver to Discord via gateway. Keep to 400 words.
"
```

## Required Data

Create `~/.hermes/data/wallets.json`:
```json
{
  "wallets": [
    {
      "label": "Main ETH",
      "address": "0x...",
      "type": "ethereum"
    },
    {
      "label": "DeFi Core",
      "address": "0x...",
      "type": "ethereum"
    }
  ]
}
```

## Tools/MCPs Needed

- **DeFiLlama MCP** or API — for DeFi position aggregation
- **Etherscan API** — for on-chain balances (or use a indexer like Alchemy/Infura)
- **Price feed** — CoinGecko API (free tier, 10-50 calls/min)

## Sample Output

```
PORTFOLIO REPORT — April 14, 2026

TOTAL: $42,350 (-$820, -1.9% 24h)

WALLET: Main ETH (0x...)
  ETH: 12.4 ETH @ $3,240 = $40,176 (+1.2%)
  USDC: $1,240 (neutral)

DEFI: $960
  Aave: 1,000 USDC deposited @ 4.2% APY = $1,020 (+0.01%)
  Uniswap: 0.5 ETH/1,600 USDC LP @ 12% APY = $3,040 (+2.1%)

ALERTS:
  ⚠️ ETH +1.2% vs BTC +0.8% — outperforming
  ✅ No drawdowns requiring action

No trades recommended. Monitor ETH resistance at $3,400.
```

---

## Feedback Loop

- "Too much data" → Hermes shows top 5, hides small positions
- "Want gas prices" → Hermes adds gas oracle check
- "Need to rebalance" → Hermes calculates target vs actual allocation

---

*Last updated: 2026-04-14*
