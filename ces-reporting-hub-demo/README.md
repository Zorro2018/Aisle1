# CES Reporting Hub — AISLE One (Portable Demo)

A fully self-contained, offline-friendly snapshot of the real
`CES-Reporting-HUB` app, built for presentations/demos. No backend, no
database, no auth, no build step — open `index.html` straight from disk
(or serve the folder with any static file server) and everything works.

## What this is (and isn't)

- **Is:** a static HTML/CSS/vanilla-JS site that mirrors the look, layout,
  and filter behavior of the real AISLE One hub's locally-rendered pages.
- **Isn't:** connected to BigQuery, Azure SQL, PingFederate SSO, or any
  Walmart system. Every number on every page is **synthetic** — generated
  in-browser by a seeded pseudo-random engine (`assets/data.js`), not
  copied from the source app's real literals. Nothing here is real
  Walmart traffic, cost, revenue, or personnel data.

## Why it looks the way it does

The real repo (`frontend/src/pages/...`) already ships small built-in
"demo mode" fallback datasets for when its backend is unreachable — those
gave us the right shape (verticals, queues, KPI names, color palette:
Bentonville Blue `#001E60` / Walmart Yellow `#FFC220` / CES Blue `#0071CE`).
This demo keeps that shape and silhouette but regenerates every value with
a seeded PRNG (`mulberry32`) keyed off the active filter combination, so:

- Numbers are different from the source app (per your "modify the numbers
  for presentation" ask).
- Changing a filter (fiscal year, vertical, team, sprint, channel, region…)
  deterministically changes the chart/table/KPI output, so filters *look
  and feel* functional in a demo without shipping a large pre-baked dataset.
- The whole dataset engine is ~150 lines — "small and efficient" by design.

## Pages included (ported from the real, locally-rendered routes)

| Page | Mirrors real route | Notes |
|---|---|---|
| `index.html` | `/` (Landing) | Hub tile grid + welcome animation |
| `pages/core-metrics.html` | `/core-metrics` (condensed) | CSAT, NPS, SLA, repeat contact rate, transfer rate, CPO - scorecard + by-vertical tabs |
| `pages/wise-queue-summary.html` | WISE's Queue Level Summary (QLS) module | Executive KPIs, daily trend, queue detail (sublob/vertical/channel), period comparison |
| `pages/billable-contacts.html` | `/billable-contacts` | Top-5 tiles, stacked volume chart, contact table |
| `pages/aht-insights.html` | `/aht-insights` | AHT KPI tiles + 12-month trend |
| `pages/monthly-billable.html` | `/monthly-billable` | Monthly totals chart + detail table |
| `pages/sprint-command-center.html` | `/executive-recap` (condensed) | Sprint KPIs, portfolio health donut, blockers |
| `pages/leadership-insights.html` | `/leadership-insights/*` (condensed) | Volume Distribution + Self-Serve tabs |
| `pages/legal.html` | `/hub/legal` | Spark Driver + Customer Feedback lookup demo |
| `pages/innovation-portal.html` | `/hub/tools/innovation` (condensed) | Idea pipeline, filter + upvote + submit |
| `pages/gamification.html` | `/hub/gamification` | Leaderboard + mini trivia quiz |

Tiles like VOX and CRM are **external** in the real app (they route to
separately-deployed repos with their own auth/CI) — they're shown on the
landing page as disabled/labeled tiles rather than ported, since there's
nothing locally-rendered here to demo. Core Metrics *is* ported in (above),
since it was a locally-rendered page in the real app rather than a separate
deployment.

WISE is *also* a genuinely separate, separately-deployed repo in the real
app (own frontend bundle, own auth, own CI — the main CES app's source even
has a regression test asserting WISE routes are *absent* from it). Unlike
VOX/CRM, though, we were handed WISE's actual frontend + backend source
separately, so `pages/wise-queue-summary.html` is a genuine (sanitized) port
of its one real module, **Queue Level Summary (QLS)** — not a guess. The
8-KPI list (Offered, Handled, SLA%, Offered%, AHT, Abandoned, ABN%, ASA),
the sum-then-ratio formulas (never average child percentages), the SLA%/ABN%
traffic-light thresholds, and the channel taxonomy (Voice/Chat/Email/
Outbound/Offline/Self Service) are all transcribed from the real
`services/queue_summary/builders/*.py` and `queueSummaryFormat.js`. The one
thing deliberately left out: the real warehouse project/dataset/table names
and the production BigQuery SQL shown in its own "Definitions & SQL"
dialog — those are internal infra identifiers, not demo material, so this
page's Definitions panel is metric text only. The page says all of this
explicitly at the top.

`Settings` and the full `Executive Recap` / `Innovation Portal` admin
tooling were intentionally condensed rather than pixel-ported — the
source files are 90–370KB of React with live-data wiring, auth checks,
and admin-only panels that don't serve a presentation demo.

## The welcome animation

On first load, `assets/welcome.js` draws "Aisle1" by hand (an SVG `<text>`
animated with `stroke-dashoffset`, like a pen tracing the letters), then
splits the banner into two jagged halves (CSS `clip-path`) that tear apart
and fly off screen, exposing the hub underneath — Apple-reveal style.

- Click, tap, or press any key to skip ahead.
- It replays on every landing-page load (by design, for repeat demos) —
  there's also a **"Replay welcome animation"** link at the bottom of the
  landing page in case you skip past it and want to show it off again.
- Respects `prefers-reduced-motion` (skips straight to content).

## Running it

Just double-click `index.html`, or for best results (so relative asset
paths and CDN fonts/Chart.js resolve consistently), serve it locally:

```bash
cd ces-reporting-hub-demo
python3 -m http.server 8787
# open http://localhost:8787
```

Requires internet access only for two CDN includes: Google Fonts
("Caveat" handwriting font) and Chart.js. Both degrade gracefully if
offline (system cursive font substitutes; chart pages would need Chart.js
to render, everything else still works).

## File map

```
index.html                       Landing page + welcome reveal
assets/style.css                 Shared design tokens + components (light/dark)
assets/data.js                   Seeded fake-data engine (no backend needed)
assets/nav.js                    Shared header / sidebar / filter-bar chrome
assets/search.js                 Global metrics search + AI strategic guidance panel
assets/welcome.js                Handwrite + tear-apart intro animation
pages/*.html                     One page per dashboard
```
