# Paper box for the official Hyperliquid SDK

A paper box is a simulated Hyperliquid account with $10,000 and live prices, reached through the official SDK's `base_url`. It fills against Hyperliquid's live book with Hyperliquid's fees and funding, and never places a real order. It is the way to test a bot without a testnet faucet, which needs a mainnet deposit first.

## Create one

```bash
curl -X POST https://api.ironflow.sh/v1/ironbox -H "Content-Type: application/json" \
  -d '{"name": "my-bot", "mode": "paper"}'
```

- No key: a trial box. It is removed after 24 hours, or after 2 hours with no requests and no strategy running. A few per IP at once.
- With a key (`Authorization: Bearer if_...`, free from https://ironflow.sh/key): the box stays.
- The answer has `base_url` (`https://api.ironflow.sh/box/ibxs_...`), shown once. It works like a password: anyone with it can trade the box, and only the box. Keep it out of logs and commits.

## Use it

```python
from eth_account import Account
from hyperliquid.exchange import Exchange
from hyperliquid.info import Info

wallet = Account.create()   # save it: the box binds to the first wallet that signs
info = Info(base_url, skip_ws=True)
exchange = Exchange(wallet, base_url=base_url)

print(exchange.market_open("BTC", True, 0.001))
print(info.user_state(wallet.address)["marginSummary"])
```

- `skip_ws=True` always. A box has no WebSocket, and without it the SDK waits on one forever.
- Signing: because `base_url` is not `https://api.hyperliquid.xyz`, the SDK signs for testnet, and a box accepts that.
- One wallet per box. A later run with a new `Account.create()` gets `User or API Wallet 0x... does not exist`. Keep the key, or derive it from something stable such as the box URL.
- `/info` answers: meta, spotMeta, allMids, l2Book, metaAndAssetCtxs, clearinghouseState, openOrders, frontendOpenOrders, orderStatus, userFills, userFillsByTime, userFunding.
- `/exchange` takes: order, cancel, cancelByCloid, modify, batchModify. Main perps, cross margin.
- No `candleSnapshot` on a box. Read candles from `GET https://api.ironflow.sh/v1/candles`.
- An order can come back with `Paper market data unavailable.` when the box's book is a few seconds old. Retry after 2 seconds.

## Strategies that run on the box

`POST {base_url}/strategies` runs a strategy around the clock with no client connected, authorised by the box URL:

- `breakout`: `size_usd`, `lookback_bars`, `exit_bars`, `bar_seconds`
- `grid`: `lower_px`, `upper_px`, `grids`, `size_usd`
- `dca`: `tranche_usd`, `budget_usd`, `dip_pct`, `interval_minutes`, `take_profit_pct`, `repeat`

Every kind needs `market` and `max_loss_usd`: at that net loss it closes and stops. Up to 3 per box, one per market. `GET {base_url}/strategies` lists them, `DELETE {base_url}/strategies/{id}` stops one, and `GET {base_url}/activity` shows what the box and its strategies did.

Guide: https://ironflow.sh/guides/hyperliquid-paper-trading. Template: the `hyperliquid-trading-bot` repo. MCP: `create_ironbox`, `start_box_strategy`, `get_box_activity` on `https://mcp.ironflow.sh/mcp`.
