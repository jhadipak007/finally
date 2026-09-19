# FinAlly API Contract

This is the shared contract between the frontend and the backend. Rules and validation are in `PLAN.md` §8.

- All endpoints are same-origin, under `/api`
- Timestamps are ISO 8601 UTC strings, except in the SSE stream, which uses Unix seconds
- Errors: `{"detail": "message"}` with 400 (validation or business rule), 404 (not found) or 503 (LLM not configured)

## Market Data

### `GET /api/stream/prices` (SSE)

The stream first sends `retry: 1000`. After that it sends one event whenever prices change (checked every ~500ms), covering all tracked tickers:

```
data: {"AAPL": {"ticker": "AAPL", "price": 190.5, "previous_price": 190.42, "open_price": 189.8, "timestamp": 1758240000.12, "change": 0.08, "change_percent": 0.042, "direction": "up"}, "GOOGL": {...}}
```

`direction` is `"up"`, `"down"` or `"flat"`. `change` and `change_percent` are vs. the previous tick.

## Portfolio

### `GET /api/portfolio`

```json
{
  "cash_balance": 8095.0,
  "total_value": 10012.4,
  "unrealized_pnl": 12.4,
  "positions": [
    {
      "ticker": "AAPL",
      "quantity": 10,
      "avg_cost": 190.5,
      "current_price": 191.74,
      "market_value": 1917.4,
      "unrealized_pnl": 12.4,
      "pnl_percent": 0.65
    }
  ]
}
```

`current_price` falls back to `avg_cost` if the cache has no price yet.

### `POST /api/portfolio/trade`

Request:
```json
{"ticker": "AAPL", "side": "buy", "quantity": 10}
```

Response 200: the executed trade plus the updated portfolio (same shape as `GET /api/portfolio`):
```json
{
  "trade": {"ticker": "AAPL", "side": "buy", "quantity": 10, "price": 190.5, "executed_at": "2026-09-19T14:03:11Z"},
  "portfolio": { "...": "same shape as GET /api/portfolio" }
}
```

Errors (400):
- `{"detail": "Insufficient cash"}`
- `{"detail": "Insufficient shares"}`
- `{"detail": "Quantity must be greater than 0"}`
- `{"detail": "Invalid ticker"}`
- `{"detail": "PYPL is not on the watchlist; add it first"}`
- `{"detail": "Price for PYPL not yet available"}`

### `GET /api/portfolio/history`

Snapshots from the last 24 hours, oldest first:
```json
[
  {"total_value": 10000.0, "recorded_at": "2026-09-19T14:00:00Z"},
  {"total_value": 10012.4, "recorded_at": "2026-09-19T14:00:30Z"}
]
```

## Watchlist

### `GET /api/watchlist`

`price`, `previous_price` and `open_price` are `null` if the ticker isn't priced yet:
```json
[
  {"ticker": "AAPL", "price": 190.5, "previous_price": 190.42, "open_price": 189.8, "added_at": "2026-09-19T13:59:00Z"}
]
```

### `POST /api/watchlist`

Request:
```json
{"ticker": "pypl"}
```

Response 201 (the ticker is uppercased):
```json
{"ticker": "PYPL", "price": 68.12, "previous_price": 68.12, "open_price": 68.12, "added_at": "2026-09-19T14:05:00Z"}
```

Errors: 400 `{"detail": "Invalid ticker"}`, 400 `{"detail": "PYPL is already on the watchlist"}`.

### `DELETE /api/watchlist/{ticker}`

Response 204. Error: 404 `{"detail": "PYPL is not on the watchlist"}`.

## Chat

### `POST /api/chat`

Request:
```json
{"message": "Add PYPL and buy 5 shares"}
```

Response 200:
```json
{
  "message": "Added PYPL to your watchlist and bought 5 shares.",
  "actions": {
    "watchlist_changes": [
      {"ticker": "PYPL", "action": "add", "status": "ok", "error": null}
    ],
    "trades": [
      {"ticker": "PYPL", "side": "buy", "quantity": 5, "price": 68.12, "status": "ok", "error": null}
    ]
  },
  "created_at": "2026-09-19T14:06:00Z"
}
```

`action` is `"add"` or `"remove"`. `status` is `"ok"` or `"failed"`; failed entries carry an `error` string and a `null` price.

Error: 503 `{"detail": "Chat unavailable: set OPENROUTER_API_KEY or LLM_MOCK=true"}`.

### `GET /api/chat/history`

Returns the last 50 messages, oldest first, so the chat panel can restore on page load:
```json
[
  {"role": "user", "content": "Add PYPL and buy 5 shares", "actions": null, "created_at": "2026-09-19T14:05:59Z"},
  {"role": "assistant", "content": "Added PYPL ...", "actions": { "...": "as above" }, "created_at": "2026-09-19T14:06:00Z"}
]
```

## System

### `GET /api/health`

```json
{"status": "ok"}
```
