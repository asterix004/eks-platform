# ADR-0016: FastAPI stock dashboard fed by Polygon.io end-of-day data

- **Status:** Accepted
- **Date:** 2026-10-18

## Context

The demo needs a believable three-tier application that is light on data,
has real read traffic for load testing, and does enough work per request that
load actually drives CPU-based autoscaling. It should also have one genuine
external secret, so that secrets handling is demonstrated on something real.

## Decision

| Component | Technology | Role |
|---|---|---|
| `web` (presentation) | FastAPI + Jinja2 templates + HTMX | Dashboard of ~50 tickers (last close, % change, sparkline, sector filter) and a detail page per ticker. **Every page shows the region, AZ, node and pod that served it**, so failover is visible in the browser. |
| `api` (application) | FastAPI, async SQLAlchemy + asyncpg, IAM DB auth | `/stocks`, `/stocks/{ticker}`; computes moving averages, RSI and volatility from ~1 year of daily bars per request (CPU-bound on purpose). |
| `ingest` (batch) | Kubernetes CronJob, Python | After US market close, fetches end-of-day bars from Polygon.io and upserts by `(ticker, date)`. **First checks whether its local Aurora instance is the writer and exits if not** — so it runs in both regions, writes only in the active one, and follows a failover automatically. |
| Data | Aurora PostgreSQL | `tickers`, `daily_bars` (~12,600 rows for a year of 50 tickers) |

- Python 3.13, dependencies managed with `uv`.
- The ticker list (~50 names across all 11 GICS sectors, from the S&P 500 and
  Nasdaq-100) lives in `app/config/tickers.yaml`.
- Polygon.io's **grouped daily** endpoint returns end-of-day bars for all US
  stocks in one call, so a daily run is a single request. A one-time backfill
  loads a year of history.

## Consequences

- The Polygon.io API key is the project's one external secret, handled per
  [ADR-0006](0006-no-static-secrets.md). The key is never shared with tooling
  or written to a file.
- `[VERIFY]` Free-tier rate limits and history depth at Polygon.io before
  writing the backfill.
- Free market-data tiers generally restrict **redistribution**. This demo
  shows end-of-day data on a short-lived site; check the provider's terms
  before making it a permanent public site. Fallback: a fixed historical
  dataset bundled with the app.
- No cache tier (ElastiCache) — kept out on purpose so load reaches the app
  pods and drives scaling.
