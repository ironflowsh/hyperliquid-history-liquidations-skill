# Hyperliquid history and liquidations skill

An Agent Skill that teaches coding agents where Hyperliquid data lives past the limits of Hyperliquid's own API: liquidations across all wallets, the liquidation map, alerts, candles past 5,000, wallet fills past 10,000 with PnL, funding and open interest history, and read-only SQL over every fill since May 2025. It ships with Ironflow's hosted MCP server. No key needed to start; a free key raises the limits.

The skill is in [`skills/hyperliquid-history-liquidations`](skills/hyperliquid-history-liquidations/SKILL.md). It follows the open [Agent Skills](https://agentskills.io) format.

## Install

**Any agent** (Claude Code, Cursor, Codex, Gemini CLI and others), with [skills](https://skills.sh):

```bash
npx skills add ironflowsh/hyperliquid-history-liquidations-skill
```

**Claude Code plugin** (the skill plus the MCP server):

```text
/plugin marketplace add ironflowsh/hyperliquid-history-liquidations-skill
/plugin install ironflow-hyperliquid@ironflow
```

**Cursor plugin** (the skill and the MCP server): install from the Cursor marketplace. To try it locally, copy this folder to `~/.cursor/plugins/local/ironflow-hyperliquid` and reload the window.

**By hand:** copy `skills/hyperliquid-history-liquidations` into `.claude/skills/`, `.cursor/skills/` or `.agents/skills/`.

## What the agent learns

- Keyless curl first: `curl "https://api.ironflow.sh/v1/liquidations?market=BTC-PERP&order=desc&limit=5"`.
- History past the caps: REST by access level, read-only SQL and free monthly Parquet files since May 2025.
- Liquidations: rows, totals, the live stream, the liquidation map and alerts to Discord, Slack, Telegram or a webhook.
- Wallets: fills with fee and realized PnL, summaries, PnL series, funding paid and positions at a past time.
- SQL: one read-only ClickHouse query over `hl.*` tables, with the schema at `/v1/query/schema`.
- Limits: keyless 30 requests a minute and 100 credits a day over 24 hours; a free key (https://ironflow.sh/key) gives 60 a minute, 1,000 credits a day and 14 days.

## Contents

| Path | For |
|---|---|
| `skills/hyperliquid-history-liquidations/` | The skill: `SKILL.md` plus `references/` |
| `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `.mcp.json` | Claude Code plugin and marketplace |
| `.cursor-plugin/plugin.json`, `mcp.json` | Cursor plugin |
| `agents-md-snippet.md` | A block to paste into a project's AGENTS.md |

The plugin connects to `https://mcp.ironflow.sh/data`: read-only data tools, no key needed.

## Services it uses

- `mcp.ironflow.sh`: Ironflow's hosted MCP server, read-only data tools.
- `api.ironflow.sh`: Ironflow's REST API, for the curl and Python examples in the skill.
- `huggingface.co`: free Parquet files of Hyperliquid history, downloaded when a task needs older data.

Requests carry the market names, time ranges, wallet addresses or SQL the user asks about. The plugin reads no local files or credentials and stores nothing on the machine. Ironflow's request logs are kept for about 90 days: https://ironflow.sh/privacy.

## Links

- [Ironflow API docs](https://docs.ironflow.sh)
- [MCP setup for every client](https://docs.ironflow.sh/guides/ai-agents)
- [Privacy](https://ironflow.sh/privacy) and [terms](https://ironflow.sh/terms)

## License

MIT
