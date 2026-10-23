---
name: scrapling-data
description: Buy live crypto and on-chain data per call over x402 (USDC) from the Scrapling Data Store at scraplingdata.xyz. Use when an agent needs a token scam/honeypot check, a go/caution/avoid token verdict, whale exchange netflow or large whale orders, who a wallet is (exchange/whale/trader/fresh), Solana token holder concentration, coin prices, DeFi yields/TVL, Polymarket odds, price-move alerts, ad cost benchmarks, or any web page as clean Markdown. No account or API key; calls cost $0.001-$0.02 and are not charged on failure.
license: MIT
metadata:
  version: "1.0.0"
  homepage: https://scraplingdata.xyz
  openapi: https://scraplingdata.xyz/openapi.json
  mcp: https://scraplingdata.xyz/mcp
---

# Scrapling Data Store (x402)

Base URL: `https://scraplingdata.xyz`. Every endpoint is a plain HTTPS GET returning JSON.
Paid calls use the x402 protocol: no signup, pay per call in USDC.

## Start free
Try these first (no payment, 30 calls/min per IP):
- `GET /v1/free/crypto/top10`: top 10 coins, price, market cap, 24h change
- `GET /v1/free/defi/top-yields`: top 5 risk-adjusted DeFi yields
- `GET /v1/free/pulse.json`: hourly whale-move summary + BTC/ETH price
- `GET /v1/free/feeds`, `GET /v1/datasets`: catalog of every feed with prices
- `GET /v1/datasets/{id}/preview`: up to 10 real rows of any dataset
- Full machine-readable list: `/openapi.json`, `/llms.txt`, `/.well-known/x402`

## Pick an endpoint
| Need | Call | Price |
|---|---|---|
| Is it safe to pay or trade with this wallet or token? safe/caution/block + 0-100 score + reasons (ethereum, base, solana) | `/v1/check/counterparty?chain=ethereum&address=0x...` | $0.02 |
| Should I trade this token? go/caution/avoid + liquidity, holders, taxes, contract flags (base, ethereum, bsc, arbitrum, polygon, optimism, avalanche, solana) | `/v1/token/verdict?chain=base&address=0x...` | $0.005 |
| Is a Base token a scam/honeypot? 0-100 risk score | `/v1/lookup/token-risk/{address}` | $0.005 |
| Whale transfers into/out of exchanges (inflow bearish, outflow bullish) + Hyperliquid whale perp flow | `/v1/signals/whale-netflow?chain=all&window=24h` | $0.02 |
| Large Hyperliquid whale orders, net buy/sell per coin | `/v1/whales/moves?min_usd=100000&window=1h` | $0.005 |
| Whale flow for one coin + mark price, funding, OI | `/v1/whales/token?chain=hyperliquid&coin=SOL` | $0.005 |
| Who is this wallet? label + 0-100 activity score, balances (ethereum, base, solana) | `/v1/wallet/profile?chain=ethereum&address=0x...` | $0.005 |
| Holder concentration of a Solana token (top-10 share, exchange-held, low/medium/high) | `/v1/token/holders?chain=solana&address={mint}` | $0.005 |
| Coin price, market cap, 24h change | `/v1/lookup/coin/bitcoin` | $0.001 |
| Coins (and DeFi TVL) that moved > X% | `/v1/alerts/price-moves?threshold=5&window=24h` | $0.002 |
| Whole market in one call | `/v1/bundle/market-pulse` | $0.005 |
| DeFi bundle: TVL, movers, stablecoins, best yields | `/v1/bundle/defi-intel` | $0.003 |
| Protocol TVL + 1d/7d trend | `/v1/lookup/protocol/aave` | $0.001 |
| Best yields for a chain/token | `/v1/lookup/yields?chain=Base&token=USDC` | $0.002 |
| Polymarket odds for an event | `/v1/lookup/polymarket?q=bitcoin` | $0.005 |
| Ad CPM/CPC/CTR benchmarks | `/v1/ads/cost-benchmarks?platform=meta&industry=retail` | $0.002 |
| Any web page as clean LLM-ready Markdown | `/v1/scrape/text?url=https://...` | $0.003 |
| Whole feed (500 coins, 1000 pools, ...) | `/v1/datasets/{id}/buy` | $0.01-$4.99 |
| Test your payment setup | `/v1/ping` | $0.001 |

Prefer the single-item lookup over buying a whole feed. Poll alerts/whale endpoints no faster than every few minutes; feeds refresh about every 30 min.

## How to pay (x402 v2)
1. Call the endpoint with no payment. You get **HTTP 402** with a `PAYMENT-REQUIRED` header (base64 JSON, same data in the body) listing `accepts` options.
2. Choose one option (scheme `exact`, USDC): Base `eip155:8453` (preferred), Polygon `eip155:137`, Arbitrum `eip155:42161`, or Solana `solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp`. Always use the `payTo`, `asset` and `amount` from the 402 response itself; do not hard-code them. The facilitator pays gas.
3. Sign the payment with an x402 client (e.g. `@x402/fetch`, `x402-axios`, Python `x402`, Coinbase AgentKit) and retry the same request with the `PAYMENT-SIGNATURE` header (`X-Payment` also accepted).
4. A `200` carries the data plus a `PAYMENT-RESPONSE` header with the settlement receipt. Payment settles only after a successful, non-empty result.

```bash
curl -si "https://scraplingdata.xyz/v1/token/verdict?chain=base&address=0x4ed4e862860bed51a9570b96d89af5e1b0efefed"   # -> 402 + PAYMENT-REQUIRED
```
```ts
import { wrapFetchWithPayment } from "@x402/fetch"; // wallet = your viem/solana signer
const pay = wrapFetchWithPayment(fetch, wallet);
const r = await pay("https://scraplingdata.xyz/v1/signals/whale-netflow?chain=all&window=24h");
console.log(await r.json());
```

Trial codes: if the user gave you a trial code, send it as the `X-Trial-Code: <code>` header (or `?trial=<code>`) to skip payment. Each code has a small number of calls and only successful calls use one. Codes are handed out by the store owner; never guess one.

MCP alternative: remote MCP server at `https://scraplingdata.xyz/mcp` exposes the same tools (pay via `params._meta['x402/payment']`).

## Errors
- `402`: payment needed or the payment was invalid/expired. Read `error`/`verificationError`, re-sign with a fresh payload, retry once. Do not loop.
- `400` / `422`: bad or missing parameter (e.g. wrong chain name, malformed address). Fix the input; no charge.
- `404`: unknown coin/token/market or nothing matched. Not charged. Try the free catalog to find valid ids.
- `429`: rate limit (free routes 30/min per IP). Back off and retry later.
- `5xx`: upstream source failed. Not charged. Retry after a short delay, at most twice.
- Check the response's freshness fields (`updated_at`, `as_of`) before acting on time-sensitive data. Data is informational, not financial advice.
