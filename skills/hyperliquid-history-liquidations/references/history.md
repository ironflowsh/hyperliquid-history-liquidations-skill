# History past the API caps

## Hyperliquid's own limits

- `candleSnapshot`: the most recent 5,000 candles per interval (about 3.5 days at 1m, 208 days at 1h).
- `userFills`: a wallet's last 2,000 fills; `userFillsByTime`: up to 10,000.
- No liquidations request, no history of other wallets' liquidations.
- Older data otherwise sits in requester-pays S3 buckets (`hyperliquid-archive`, `hl-mainnet-node-data`).

## Ironflow REST, by access

Base URL `https://api.ironflow.sh`. Fill history starts on 25 May 2025.

| Access | Requests | History |
|---|---|---|
| No key | 30 a minute, 100 credits a day per IP | 24 hours |
| Free key (https://ironflow.sh/key) | 60 a minute, 1,000 credits a day and 3,000 a month | 14 days |
| Plans: Explorer $49, Builder $149, Pro $499 a month | from 100 a minute | 90 days, 1 year, every fill since May 2025 |

All plan limits: https://docs.ironflow.sh/concepts/rate-limits. A `from` older than your reach is cut to it, with `X-Ironflow-History-Clamped` (Unix ms) and an `X-Ironflow-Notice` that says so.

| Data | Call |
|---|---|
| Candles 1m, 5m, 15m, 1h, 4h, 1d | `GET /v1/candles?market=BTC-PERP&interval=1h&from=-7d&limit=1000` |
| All trades in a market | `GET /v1/trades?market=BTC-PERP&from=-1h` |
| One wallet's fills, with fee and realized PnL | `GET /v1/fills?address=0x...&from=-7d` |
| Liquidations | `GET /v1/liquidations?market=BTC-PERP&from=-24h` |
| Funding | `GET /v1/funding?market=BTC-PERP` |
| Open interest | `GET /v1/open-interest?market=BTC-PERP` |
| Mark and oracle prices | `GET /v1/mark-prices?market=BTC-PERP` |
| A wallet's positions at a past time | `GET /v1/positions?address=0x...&at=<ISO time inside your history>` |

Paging: `limit` up to 1000, then pass `pagination.next_cursor` as `cursor` until `pagination.has_more` is false. `order=desc&limit=1` is the latest row. Full spec: https://api.ironflow.sh/openapi.yaml.

## SQL

`GET` or `POST https://api.ironflow.sh/v1/query` with `sql`: one read-only ClickHouse SELECT over `hl.*` tables (`hl.fills`, `hl.trades`, `hl.liquidations`, `hl.market_hourly`, `hl.funding`, `hl.open_interest`, `hl.mark_prices`, `hl.wallet_daily`, ...). Tables and columns: `GET /v1/query/schema`. Keyless: 1 day back, 5 s and 1,000 rows a query, 1 minute of query time an hour. A free key: 30 days, 10 s, 1,000 rows. A query costs 10 credits. Aggregate in SQL; for totals over days or months read `hl.market_hourly`.

```bash
curl -G https://api.ironflow.sh/v1/query --data-urlencode \
  "sql=SELECT toStartOfHour(ts) AS hour, sumIf(notional, side = 'sell') AS longs_usd FROM hl.liquidations WHERE market = 'BTC-PERP' AND ts >= now() - INTERVAL 24 HOUR GROUP BY hour ORDER BY hour"
```

## Free monthly files

Parquet files on Hugging Face (`https://huggingface.co/datasets/ironflow/<dataset>/resolve/main/<file>`):

| Dataset | File | From | One row per |
|---|---|---|---|
| `hyperliquid-liquidations` | `data/events/YYYY-MM.parquet` | 2025-05 | liquidated position (`timestamp`, `market`, `position_side`, `notional_usd`, `avg_price`) |
| `hyperliquid-liquidations` | `data/fills/YYYY-MM.parquet` | 2025-05 | liquidation fill |
| `hyperliquid-candles-1h` | `data/YYYY-MM.parquet` | 2025-05 | market and hour (`timestamp`, `market`, `coin`, `open`, `high`, `low`, `close`, `volume`, `trades`) |
| `hyperliquid-funding` | `data/YYYY-MM.parquet` | 2023-05 | market and funding hour |
| `hyperliquid-open-interest` | `data/YYYY-MM.parquet` | 2023-05 | market and time |
| `hyperliquid-trades` | `data/YYYY-MM-DD.parquet` | see the card | trade |

```python
import pandas as pd  # pip install pandas pyarrow

url = "https://huggingface.co/datasets/ironflow/hyperliquid-candles-1h/resolve/main/data/2026-10.parquet"
btc = pd.read_parquet(url, filters=[("market", "==", "BTC-PERP")])
```

From 10 November 2026 new files appear 90 days after the data date, carry no wallet addresses and are for non-commercial use (CC BY-NC 4.0). Use the API for recent days, wallet-level data or commercial use. Each dataset card states its license.

## Hyperliquid SDK as a client

With a key, `https://api.ironflow.sh/rpc/<key>/info` takes the same body as Hyperliquid's `/info` (clearinghouseState, openOrders, meta, allMids, l2Book, candleSnapshot and more) on the key's limits, so `base_url` in a Hyperliquid SDK changes the endpoint. No `/exchange` and no WebSocket there. Guide: https://docs.ironflow.sh/guides/hyperliquid-rpc
