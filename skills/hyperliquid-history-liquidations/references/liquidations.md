# Liquidations, the map and alerts

Ironflow reads the `liquidation` field Hyperliquid puts on every liquidation fill, from its own nodes. Rows are fills Hyperliquid itself marked, not a heuristic.

## Rows: GET /v1/liquidations

```bash
curl "https://api.ironflow.sh/v1/liquidations?market=BTC-PERP,ETH-PERP&from=-6h&limit=1000"
```

- `market`: one, a comma list of up to 50, a coin (`BTC`, `xyz:NVDA`), or omit for every market.
- `address`: one wallet. `from`, `to`: window. `order`: `asc` (default) or `desc`. `limit` up to 1000, `cursor` for the next page.
- New rows only: pass the response's `poll_cursor` as `after`. Rows up to `complete_until` are final.
- Each row: `timestamp` (Unix ms), `market`, `side`, `price`, `size`, `address`, `realized_pnl`, `leverage_type`, `bankrupt`, `block_number`, `hash`.
- `side` is the liquidated wallet's closing fill: `sell` closes a long, `buy` closes a short. One liquidation often fills in several rows.
- Keyless rows end 60 s before now. A free key reads them live.

## Totals: GET /v1/liquidations/summary

`?window=1h` (15m, 1h, 4h, 12h, 24h keyless; 7d with a key) and optional `market`. Returns count, wallets, long and short USD, the top markets and the largest liquidation, computed over every row.

## Live: GET /v1/liquidations/stream

Server-Sent Events, one row per event in the same shape as `/v1/liquidations`. Filters: `market`, `address`, `min_usd`.

```bash
curl -N "https://api.ironflow.sh/v1/liquidations/stream?market=BTC,ETH&min_usd=10000"
```

- Events: `event: liquidation` with a JSON `data` line, `: ping` every 5 s.
- `event: reconnect` comes after an hour or a deploy. Reconnect with `Last-Event-ID` (or `?last_event_id=`) and the missed rows of the last 5 minutes come first.
- One stream per IP without a key; a free key holds 2. Opening a stream does not count toward the keyless daily limit.

## The map: GET /v1/liquidations/map

Where open perp positions would be liquidated if the price moved: longs below the mark, shorts above. Built from every open position in the node's state snapshot, about every 12 minutes.

```bash
curl "https://api.ironflow.sh/v1/liquidations/map?market=BTC-PERP&range_pct=10"
```

- Parameters: `market` (required), `bucket_size` (`auto` or a price width), `range_pct` (up to 50), `side` (`both`, `long`, `short`).
- Each level: price range, notional, position count, and `cum_long_usd` / `cum_short_usd`, everything liquidated between the mark and that level.
- Keyless: the 10 largest levels within 10% of mark. Any key returns every level.
- `GET /v1/liquidations/map/markets` lists the markets. Past liquidations by price are at `GET /v1/analytics/liquidation-levels?market=`.

## Alerts: POST /v1/alerts (needs a key)

Ironflow posts matching liquidations to a channel within seconds, at most one message every 5 seconds per alert.

```bash
curl -X POST https://api.ironflow.sh/v1/alerts -H "Authorization: Bearer $IRONFLOW_KEY" \
  -H "Content-Type: application/json" \
  -d '{"url": "https://discord.com/api/webhooks/...", "markets": ["BTC", "ETH"], "side": "long", "min_usd": 100000}'
```

- Destination: `url` (a Discord or Slack webhook, or any public HTTPS URL) or `telegram_chat` (then add the bot named in the response's `setup` to the chat).
- Filters: `markets`, `side` (`any`, `long`, `short`: the liquidated position), `min_usd`, `address`, and `name` for the title.
- A plain webhook gets signed JSON. `signing_secret` appears once, in the create response.
- `POST /v1/alerts/{id}/test` sends a sample now. `GET /v1/alerts` lists them, `DELETE /v1/alerts/{id}` removes one.
- A free key allows 1 alert.

## MCP tools

On `https://mcp.ironflow.sh/mcp`: `get_liquidations`, `get_liquidation_summary`, `get_liquidation_map`, `get_liquidation_levels`, and with a key `create_liquidation_alert`, `test_liquidation_alert`, `list_liquidation_alerts`, `delete_liquidation_alert`.
