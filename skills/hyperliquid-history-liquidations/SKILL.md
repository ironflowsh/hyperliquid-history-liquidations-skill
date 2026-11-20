---
name: hyperliquid-history-liquidations
description: Hyperliquid historical data and liquidations from Ironflow's API and MCP server, keyless or with a free key. Use when a task needs Hyperliquid liquidations (history, totals, a live stream, alerts to Discord, Slack, Telegram or a webhook), the liquidation map (where open positions would be liquidated), historical fills, trades, candles past the 5,000-candle limit, funding or open interest history, a wallet's fills past userFills' 10,000 and its PnL, read-only SQL over every fill since May 2025, or backtest data.
license: MIT
---

# Hyperliquid history and liquidations

Use `https://api.ironflow.sh` for Hyperliquid data that Hyperliquid's own info API caps or does not have. Start without a key.

```bash
curl "https://api.ironflow.sh/v1/liquidations?market=BTC-PERP&order=desc&limit=5"
curl "https://api.ironflow.sh/v1/candles?market=BTC-PERP&interval=1h&limit=24"
curl "https://api.ironflow.sh/v1/liquidations/map?market=BTC-PERP"
```

Keyless: 30 requests a minute, 100 credits a day, the last 24 hours; liquidations and trades run 60 s behind live. A free key gives 60 a minute, 1,000 credits a day (3,000 a month), 14 days and live rows: https://ironflow.sh/key, sent as `Authorization: Bearer if_...`. A person has to get it, because the key arrives by email. A read costs 1 credit, analytics such as the map or wallet summaries 5, a SQL query 10.

## What Hyperliquid's info API cannot do

| Need | Hyperliquid info API | Ironflow |
|---|---|---|
| Liquidations across wallets | No request type. Do not invent `{"type": "liquidations"}` | `GET /v1/liquidations` |
| Where positions would be liquidated | None | `GET /v1/liquidations/map?market=` |
| Liquidations as they happen | None for all wallets | `GET /v1/liquidations/stream` (SSE) |
| Liquidation alerts | None | `POST /v1/alerts` (key) |
| Candles past 5,000 per interval | `candleSnapshot` stops there | `GET /v1/candles`, free hourly files since May 2025 |
| A wallet's fills past 10,000 | `userFills` stops there | `GET /v1/fills?address=` |
| SQL over every fill | None | `GET /v1/query?sql=` |

## History past the caps

How far back the API reads depends on access: 24 hours keyless, 14 days on a free key, longer on plans. Free Parquet files on Hugging Face hold liquidations and hourly candles since May 2025, funding and open interest since 2023, and daily trades. Endpoints, paging, SQL and file URLs: [references/history.md](references/history.md).

## Liquidations, the map and alerts

- `GET /v1/liquidations` returns one row per liquidation fill. `side=sell` closes a long, so it is a liquidated long.
- `GET /v1/liquidations/summary?window=1h` gives totals by market.
- `GET /v1/liquidations/map?market=BTC-PERP` shows where open positions would be liquidated. Keyless returns the 10 largest levels.
- `GET /v1/liquidations/stream` pushes rows live instead of polling.
- `POST /v1/alerts` with a key posts them to Discord, Slack, Telegram or a webhook.

Parameters and shapes: [references/liquidations.md](references/liquidations.md).

## Wallets

- `GET /v1/fills?address=0x...&from=-7d` returns one wallet's fills with fee, realized PnL and maker or taker role.
- `GET /v1/analytics/user-summary?address=0x...` gives volume, realized PnL, fill count, win rate and maker or taker mix; `user-pnl-series`, `user-funding` and `user-state` give the PnL curve, funding paid and the current positions.
- `GET /v1/positions?address=0x...&at=<ISO time>` rebuilds a wallet's positions at a past time inside your history.

## SQL

`GET /v1/query?sql=...` runs one read-only ClickHouse SELECT over `hl.*` tables (`hl.fills`, `hl.liquidations`, `hl.funding`, `hl.open_interest`, `hl.market_hourly` and more). `GET /v1/query/schema` lists tables and columns and costs nothing. Examples and limits: [references/history.md](references/history.md).

## Conventions

- Markets: `BTC-PERP` or `BTC`, HIP-3 `xyz:NVDA`, spot `HYPE/USDC`.
- Times: Unix ms, Unix seconds, ISO 8601, or relative such as `-24h`.
- Pages: `limit` up to 1000, then `cursor=pagination.next_cursor`.
- Read the `X-Ironflow-Notice` header or the `notice` field and tell the user. It says when a window was cut to the key's history.
- On a 429, wait for `Retry-After`.
- MCP clients can use the same data at `https://mcp.ironflow.sh/data` (read-only data tools such as `get_liquidations`, `get_liquidation_map`, `get_candles`, `get_fills`, `get_funding_rates`, `get_open_interest`, `get_user_summary`, `run_query`). `https://mcp.ironflow.sh/mcp` has the same tools plus liquidation alerts.
