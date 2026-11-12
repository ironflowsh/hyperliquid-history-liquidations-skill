# Hyperliquid history and liquidations skill

An Agent Skill that teaches coding agents where Hyperliquid data lives past the limits of Hyperliquid's own API: liquidations across all wallets, the liquidation map, alerts, candles past 5,000, wallet fills past 10,000, and a paper account for bots built on the official SDK. It ships with Ironflow's hosted MCP server. No key needed.

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

**Cursor plugin** (the skill, a rule and the MCP server): install from the Cursor marketplace. To try it locally, copy this folder to `~/.cursor/plugins/local/ironflow-hyperliquid` and reload the window.

**By hand:** copy `skills/hyperliquid-history-liquidations` into `.claude/skills/`, `.cursor/skills/` or `.agents/skills/`. For Cursor rules only, copy `rules/hyperliquid-data.mdc` into `.cursor/rules/`.

## What the agent learns

- Keyless curl first: `curl "https://api.ironflow.sh/v1/liquidations?market=BTC-PERP&order=desc&limit=5"`.
- History past the caps: REST by access level, read-only SQL and free monthly Parquet files since May 2025.
- Liquidations: rows, totals, the live stream, the liquidation map and alerts to Discord, Slack, Telegram or a webhook.
- Paper trading: `POST /v1/ironbox` returns a `base_url` for the official Hyperliquid SDK, with `skip_ws=True`.

## Contents

| Path | For |
|---|---|
| `skills/hyperliquid-history-liquidations/` | The skill: `SKILL.md` plus `references/` |
| `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `.mcp.json` | Claude Code plugin and marketplace |
| `.cursor-plugin/plugin.json`, `mcp.json`, `rules/` | Cursor plugin |
| `agents-md-snippet.md` | A block to paste into a project's AGENTS.md |

The plugin connects to `https://mcp.ironflow.sh/mcp?toolset=data`: read-only data tools, no key needed.

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
