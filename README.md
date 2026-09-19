# FinAlly — AI Trading Workstation

An AI-powered trading workstation with live market data, a simulated $10k portfolio, and an LLM chat assistant that can analyze positions and execute trades. It looks like a Bloomberg terminal with an AI copilot.

Built by coding agents as the capstone project for an agentic AI coding course.

## Features

- Live price streaming over SSE, with green/red price flashes and sparklines
- Market orders with instant fills, a positions table, a P&L heatmap and a portfolio value chart
- AI chat (LiteLLM → OpenRouter, Cerebras) that trades and edits the watchlist for you
- Built-in market simulator, or real data from the Massive (Polygon.io) API

## Stack

One Docker container on port 8000:

- **Frontend**: Next.js static export, TypeScript, Tailwind
- **Backend**: FastAPI, managed with `uv`
- **Database**: SQLite, created and seeded on first run

## Quick Start

```bash
cp .env.example .env          # then add OPENROUTER_API_KEY
./scripts/start_mac.sh        # Windows: ./scripts/start_windows.ps1
```

Open http://localhost:8000. Stop with `scripts/stop_mac.sh` or `scripts/stop_windows.ps1`; your data persists in the `finally-data` volume.

## Environment Variables

| Variable | Required | Description |
|---|---|---|
| `OPENROUTER_API_KEY` | For chat | OpenRouter key; without it, only `/api/chat` is unavailable |
| `MASSIVE_API_KEY` | No | Real market data; leave empty to use the simulator |
| `LLM_MOCK` | No | `true` for deterministic mock chat responses (testing) |

## Development

```bash
cd backend
uv sync --dev
uv run pytest
```

## Project Layout

```
backend/    FastAPI app, market data, database
frontend/   Next.js app
planning/   Spec (PLAN.md) and API contract (API.md)
scripts/    Start/stop scripts
test/       Playwright E2E tests
```

## Status

The market data subsystem is complete. The rest of the platform is being built in the order set out in [planning/PLAN.md](planning/PLAN.md) §13.

## License

See [LICENSE](LICENSE).
