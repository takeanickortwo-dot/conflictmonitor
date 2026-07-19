# Plan — Global Conflict & Markets Monitor (live website)

## Deliverable
A single-page live dashboard website ("war room" style) combining:
- **War / conflict monitoring**: live global news via GDELT API (CORS-friendly), conflict
  keyword feeds, news tone/intensity metrics, breaking alerts
- **Economic conflict reporting**: oil (WTI/Brent), gold, natgas, S&P 500 / Nasdaq / Dow,
  index futures, VIX, USD, big tech stocks (AAPL, MSFT, NVDA, AMZN, GOOGL, META, TSLA)

## Stage 1 — Verify live data sources (Orchestrator, direct)
- curl-test GDELT DOC 2.1 API (news, CORS headers, JSON format)
- curl-test Stooq quote CSV/JSON (market data, CORS-friendly) as primary market source
- Pull current market snapshot via yahoo_finance plugin for seed data
- Output: verified endpoint list + sample payloads + snapshot JSON

## Stage 2 — Build webapp (load `vibecoding-webapp-swarm`)
- Dispatch builder subagent with: design brief, verified endpoints, seed snapshot
- Layout: ticker tape header → market grid (energy / indices / futures / big tech) →
  live conflict news feed (GDELT) → conflict monitor stats → auto-refresh
- Aesthetic: dark war-room, muted low-saturation palette (amber/red/teal accents),
  mono/typewriter data fonts, dense info, no blue-purple gradients
- React + Vite + Tailwind (per skill), client-side fetch + 60s auto-refresh + fallback data

## Stage 3 — Validate & deliver
- Build the project, fix errors, verify with screenshot/browser
- Deliver via mshtools-website_version_manager (build_version)
