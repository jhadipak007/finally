# FinAlly — AI Trading Workstation

An AI-powered trading workstation with live market data, a simulated $10k portfolio, and an LLM chat assistant that can analyze positions and execute trades. Vision: a Bloomberg-style terminal with an AI copilot, shipped as a single Docker container.

Built entirely by coding agents as the capstone project for an agentic AI coding course. Agents coordinate through the docs in `planning/`, chiefly [`planning/PLAN.md`](planning/PLAN.md), the full project spec.

## Planned Features

- Live price streaming over SSE, with green/red price flashes and sparklines
- Market orders with instant fills, a positions table, a P&L heatmap and a portfolio value chart
- AI chat (LiteLLM → OpenRouter, Cerebras) that trades and edits the watchlist for you
- Built-in market simulator, or real data from the Massive (Polygon.io) API

## Target Stack

One Docker container on port 8000:

- **Frontend**: Next.js static export, TypeScript, Tailwind
- **Backend**: FastAPI, managed with `uv`
- **Database**: SQLite, created and seeded on first run

## Current Status

Following the build order in [`planning/PLAN.md`](planning/PLAN.md) §13:

| # | Step | Status |
|---|------|--------|
| 1 | Database + portfolio API | Not started |
| 2 | Watchlist API | Not started |
| 3 | Chat (mock, then real LLM) | Not started |
| 4 | Frontend | Not started |
| 5 | Docker + scripts | Not started |
| 6 | E2E tests | Not started |

**Market data** (a component shared across those steps, ahead of schedule) **is complete**: `backend/app/market/` implements the simulator (GBM with correlated moves) and the Massive/Polygon.io client behind a common interface, a thread-safe price cache, and the `/api/stream/prices` SSE endpoint — 73 passing tests, 84% coverage. See [`planning/MARKET_DATA_SUMMARY.md`](planning/MARKET_DATA_SUMMARY.md).

There is no FastAPI app entrypoint, database, frontend, Docker setup, or chat integration yet — none of `backend/db/`, `frontend/`, `scripts/`, `test/`, or a `Dockerfile` exist in the repo yet.

## Repo Layout (current)

```
backend/
├── app/market/     Market data subsystem (simulator, Massive client, cache, SSE stream) — done
├── tests/market/   Tests for the above
└── market_data_demo.py   Rich terminal demo of the live simulator
planning/
├── PLAN.md                  Full project spec
├── API.md                   REST/SSE contract (frontend ⇄ backend)
├── MARKET_DATA_SUMMARY.md   Summary of the completed market data component
└── archive/                 Historical planning docs
```

## Development

```bash
cd backend
uv sync --extra dev
uv run pytest              # 73 tests, market data subsystem
uv run market_data_demo.py # live terminal dashboard of simulated prices
```

## Environment Variables

Per [`planning/PLAN.md`](planning/PLAN.md) §5 (not all are wired up yet — only `MASSIVE_API_KEY` is currently read, by the market data factory):

| Variable | Required | Description |
|---|---|---|
| `OPENROUTER_API_KEY` | For chat | OpenRouter key; without it, only `/api/chat` is unavailable |
| `MASSIVE_API_KEY` | No | Real market data; leave empty to use the simulator |
| `LLM_MOCK` | No | `true` for deterministic mock chat responses (testing) |

## License

See [LICENSE](LICENSE).
