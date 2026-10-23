# Scrapling Data Agent Skill

An [Agent Skill](https://agentskills.io) that lets AI agents buy live crypto and on-chain data per call over x402 (USDC) from the [Scrapling Data Store](https://scraplingdata.xyz): token scam checks, whale flows, wallet labels, prices, DeFi yields, Polymarket odds, and web-to-Markdown. No account or API key.

- Skill: [`skills/scrapling-data/SKILL.md`](skills/scrapling-data/SKILL.md)
- Website: https://scraplingdata.xyz
- OpenAPI: https://scraplingdata.xyz/openapi.json
- MCP: https://scraplingdata.xyz/mcp

## Install
Copy `skills/scrapling-data/` into your agent's skills folder (e.g. `~/.claude/skills/`).

License: MIT


## New: Trust Bundle ($0.04)
`GET https://scraplingdata.xyz/v1/trust/agent?chain=base&address=0x...` — allow / caution / block + 0-100 trust score + reasons for any outside agent, wallet or token, in one x402 call.

## Paid MCP tools
Remote MCP server with every store endpoint as a pay-per-call tool (x402): `claude mcp add --transport http scrapling-data https://scraplingdata.xyz/mcp`. See [mcp/README.md](mcp/README.md).
