# Scrapling x402 Data Store — paid MCP tools

A remote MCP server (Streamable HTTP) with ~30 tools: crypto/DeFi lookups, whale flow, token verdicts,
wallet profiles, Counterparty Check, the **Trust Bundle** (`trust_agent`), web scraping and full datasets.
Free tools need nothing. Paid tools cost $0.001–$0.04 USDC per call, paid with **x402** — no account, no API key.

**Endpoint:** `https://scraplingdata.xyz/mcp`
**Registry name:** `io.github.bnth09productions-maker/scrapling-x402-store`
**Server card:** https://scraplingdata.xyz/mcp/server-card

## Install

Claude Code:
```
claude mcp add --transport http scrapling-data https://scraplingdata.xyz/mcp
```

Cursor / Claude Desktop / any client (`mcp.json`):
```json
{ "mcpServers": { "scrapling-data": { "url": "https://scraplingdata.xyz/mcp" } } }
```
Clients without remote support:
```json
{ "mcpServers": { "scrapling-data": { "command": "npx", "args": ["-y", "mcp-remote", "https://scraplingdata.xyz/mcp"] } } }
```

## How payment works (x402 MCP transport, coinbase/x402 `specs/transports-v2/mcp.md`)
1. Call a paid tool with no payment → result has `isError: true` and `structuredContent` = x402 `PaymentRequired`
   (`accepts`: USDC on Base `eip155:8453` preferred, also Polygon, Arbitrum, Solana; payTo `0x72C80C01fD89Bac74f47cf604b61ACd119c9BCdb`).
2. Sign an `exact` payment and retry the same call with it in `params._meta["x402/payment"]`.
3. Success returns the data plus `_meta["x402/payment-response"]` (tx hash). Failed lookups are not charged.

Works with x402-aware MCP clients (e.g. `@x402/mcp` client wrappers, Coinbase Agentic Wallet). Prepaid alternative:
pass `api_key: "sk_scrapling_..."` as a tool argument.

## Featured tool: `trust_agent` ($0.04)
"Should I trust this agent?" One call returns `allow / caution / block`, a 0-100 trust score and reasons by
combining Counterparty Check, wallet profile and (for tokens) the token security verdict. Chains: base, ethereum, solana.
Same thing over plain HTTP: `GET https://scraplingdata.xyz/v1/trust/agent?chain=base&address=0x...`

## Example (raw JSON-RPC)
```
curl -s -X POST https://scraplingdata.xyz/mcp -H 'content-type: application/json' \
  -H 'accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```
