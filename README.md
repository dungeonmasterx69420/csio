# CSIO — Counter-Strike Tier 1 · Match Control Room

Self-hosted CS2 dashboard focused on scheduling, live scores and watch links — HLTV energy, Dungeon styling. Standalone service (same architecture as CUPIO/MAGICO: Express static server + cached API proxy + built-in snapshot fallback).

## What it shows

- **Scene Wire ticker** — Cologne Major result, roster moves, upcoming circuit dates
- **LIVE NOW zone** — every tier-1 match in play with series score and a ▶ Watch button (stream links straight from the feed, usually Twitch)
- **GOAT WATCH — s1mple tracker** — BC.Game roster state, and his next match with opponent, event, live countdown and stream link the moment one is scheduled; flips to a LIVE state with score when he's playing
- **Match Board** — HLTV-style rows grouped by day: time, teams, BO format, event, tier stars, watch link
- **Recent Results** — hydrates from the feed (48h window) with per-match **Stats · HLTV** and **VOD** (Twitch channel videos) links; seeded with the Cologne Major playoff bracket
- **Tier 1 Calendar** — every big event from BLAST Bounty (Jul 20) through the PGL Singapore Major (Nov 25 – Dec 13), with Twitch channel links for the known broadcasters
- **Dungeon strip** — recruitment CTA to enterdungeon.cc

CSIO is a **PWA** (same treatment as YARDIO) — installable to the home screen, with an
offline-capable app shell and last-loaded scores served from cache when the network drops
(`manifest.webmanifest` + `sw.js`: network-first for navigations and `/api/*`, cache-first
for fonts and images).

## Data source

HLTV has no public API, so live data comes from **PandaScore**:

1. Get a free key at https://pandascore.co (free tier is plenty — the server caches)
2. Set it as `PANDASCORE_TOKEN` on the service

Endpoints proxied server-side: `/csgo/matches/running`, `/csgo/matches/upcoming`, `/csgo/matches/past` (time-bounded with `range[end_at]` to the last 48h at `per_page=100`, so busy qualifier days can't push recent tier-1 results out of the window; falls back to the plain query if the provider rejects the param). Caching: 60s while any match is live, 5 min idle, 45s error backoff, stale-serve on failure. Without a token (or if the provider is down) the API returns `{ source: 'none' }` and the frontend runs on its built-in snapshot (seeded **July 4, 2026**) — the page never breaks.

The frontend polls `/api/cs` every 60 seconds over relative paths (tunnel/proxy-safe, no WebSockets).

## Filtering & env vars

| Var | Default | Purpose |
|---|---|---|
| `PANDASCORE_TOKEN` | — | enables live data |
| `CS_TIERS` | `s,a` | which series tiers count as "tier 1" on the board |
| `CS_TEAM_REGEX` | `bc[\.\s]?game` | the tracked team — **matches for this team always pass the tier filter**, since BC.Game currently live in qualifier land. If s1mple transfers, change this one env var and the whole tracker follows him. |

## Run / Deploy

```bash
npm install
npm start          # PORT or 3000
```

Render: Build `npm install`, Start `npm start`, env var `PANDASCORE_TOKEN`. Health check: `GET /api/cs/health`. Then point `csio.enterdungeon.cc` at it via a Cloudflare CNAME to the new service's `*.onrender.com` hostname.

## Note

CSIO is an unofficial fan dashboard and is not affiliated with Valve, HLTV, or any tournament organizer.
