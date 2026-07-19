# Research — Global Conflict & Markets Monitor

## Product concept
A live "war room" dashboard: real-time conflict/war news monitoring fused with
conflict-sensitive market data (oil, gold, index futures, S&P 500, VIX, USD, big tech).
Dense, operational, data-rich. Think: a command-center terminal for geopolitical risk.

## Verified live data sources (IMPORTANT — builders must use these)

### 1. GDELT DOC 2.1 API — live conflict news (verified working, CORS open)
- Base: `https://api.gdeltproject.org/api/v2/doc/doc`
- Confirmed response header: `Access-Control-Allow-Origin: *` → safe for browser-side fetch.
- Rate limit: max 1 request / 5 seconds per client. App must throttle: fetch at most
  every ~60s per view, cache results in state/localStorage, show "as of HH:MM" timestamps.
- Article list mode:
  `?query=war%20OR%20missile%20OR%20airstrike&mode=artlist&maxrecords=25&format=json&sort=hybridrel`
  Returns `{articles:[{url, url_mobile, title, seendate, socialimage, domain, language, sourcecountry}]}`
  `seendate` format: `YYYYMMDDHHMMSS` (UTC).
- Useful preset query feeds (URL-encode!):
  - Breaking conflict: `(war OR airstrike OR missile OR drone strike OR shelling OR ceasefire)`
  - Escalation watch: `(invasion OR offensive OR mobilization OR "declares war" OR retaliation)`
  - Energy war / oil: `(oil OR OPEC OR "strait of hormuz" OR refinery OR pipeline) AND (attack OR war OR sanctions)`
  - Sanctions & economic war: `(sanctions OR embargo OR tariffs OR "trade war" OR export controls)`
  - Nuclear & high tension: `(nuclear OR ICBM OR warhead OR NATO)`
- GDELT timespan parameter: `&timespan=24h` (also 1h, 6h, 48h, 1w). Use 24h for wire.
- Also verified: `mode=timelinevol` returns volume time series usable for a
  "news intensity" sparkline: `&mode=timelinevol&format=json` →
  `{timeline:[{date, value, norm}]}` style buckets (handle shape defensively).

### 2. Seed market snapshot (build-time, embedded)
- File: `/mnt/agents/output/seed_market_data.json` (already generated).
- 21 assets from Yahoo Finance (plugin), as of 2026-07-17:
  indices (^GSPC 7457.69 -1.01%, ^IXIC 25520.24 -1.40%, ^DJI 52146.42 -0.77%, ^VIX 18.77 +12.19%),
  futures (ES=F 7497.75, NQ=F 28773.25, YM=F 52375),
  energy (CL=F WTI 81.78 +3.58%, BZ=F Brent 88.10 +4.59%, NG=F 2.91),
  metals (GC=F gold 4018.80, SI=F silver 56.33),
  fx (DX-Y.NYB dollar index 100.75),
  big tech (AAPL 333.74, MSFT 393.82, NVDA 202.81, AMZN 247.23, GOOGL 346.77,
  META 646.01, TSLA 380.84, AVGO 370.83 — most down 1-3%).
- Each asset has: ticker, name, category, unit, price, prevClose, change, changePct,
  asOf, and a `series` array (up to ~20 daily closes) for sparklines.
- JSON schema: `{asOf, generatedAt, source, assets:[...]}` — copy into
  `src/data/seedMarketData.ts` as a typed const.

### 3. Optional client-side market refresh (progressive enhancement only)
- Browser may attempt `https://stooq.com/q/l/?s=<syms>&f=sd2t2ohlcv&h&e=csv`
  (stooq symbols: ^spx, ^ndx, ^dji, cl.f, bz.f, gc.f, ng.f, es.f, nq.f, ym.f, dx.f,
  aapl.us, msft.us, nvda.us, amzn.us, googl.us, meta.us, tsla.us, avgo.us).
- MUST fail silently to seed data on any error (CORS/network). Never block render.
- All market UI must always show a visible "as of <date>" stamp.

## Functional requirements
1. **Ticker tape** header: scrolling marquee of all 21 assets with price + % change.
2. **Market grid**: grouped cards — Indices / Futures / Energy / Metals / FX / Big Tech —
   each with price, change, change%, mini sparkline from `series`, red/green direction.
3. **Conflict wire** (live GDELT): tabbed feeds (Breaking / Escalation / Energy War /
   Sanctions / Nuclear), article title, source domain, country, relative time, link out.
4. **Conflict intensity panel**: derived metrics from the wire — article count per theme,
   intensity sparkline (timelinevol), "top mentioned hotspots" parsed from titles
   (simple keyword tally: Ukraine, Russia, Iran, Israel, Gaza, Taiwan, China, Red Sea,
   Hormuz, NATO...).
5. **Markets ↔ conflict correlation strip**: highlight that oil +3.6% / VIX +12% coincides
   with escalation headlines (derived from live feed + seed data; presented as
   informational, not financial advice).
6. Auto-refresh: GDELT every 60s (respecting 5s min gap between the tab fetches —
   fetch only active tab, or sequential queue), countdown indicator, manual refresh button.
7. Status indicators: LIVE (feed ok), DELAYED (feed error, showing cache), SEED DATA badge
   on market panels.
8. Timezone-aware timestamps; dark war-room aesthetic; fully responsive.

## Non-goals
- No backend, no auth, no API keys. Pure static frontend.
- No financial advice claims; add small disclaimer in footer.
