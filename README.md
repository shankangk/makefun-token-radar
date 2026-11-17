# Token Radar — make.fun token-radar-01

[![token-radar-01](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01)
[![token-radar-01](https://make.fun/badge/token-radar-01-dev.svg)](https://make.fun/bounty/token-radar-01)
[![token-radar-01](https://make.fun/badge/token-radar-01-final.svg)](https://make.fun/bounty/token-radar-01)

A token launch radar: it reads a sealed DefiLlama snapshot, ranks every
pre-token protocol by an implied value, and hands the reader a dashboard
with a filterable board, a detail view for each protocol, and a concrete
farm plan for each one.

Bounty: https://make.fun/bounty/token-radar-01

## How it works

`main.ts` (Node 22, standard library only) reads `/input/snapshot.json`,
scores the board with `lib/model.ts` (radar-2), writes `board.json` and
`data.js` to `/output`, copies `dashboard/` and `screenshots/`, and writes
`dossier.json` with the vendored `vendor/makefun-board.ts`.

Run it locally:

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input ../input/snapshot.json
```

Then open `out/dashboard/index.html` in a browser. The page renders with
no network: all data comes from `data.js` beside it.

## The model (radar-2)

Implied value follows only from the snapshot's own figures, and no protocol
name is ever special-cased:

```
implied = TVL × 0.20 × size × growth × maturity × chains × category
```

- **base 0.20** — the live market's own anchor: across the snapshot's
  live-token protocols, the median market-cap / TVL ratio is about 0.20.
- **size 0.85–1.00** — grows with log TVL, so size alone cannot dominate.
- **growth = 1 + clamp(7d, −50, +150) / 100 × 0.75** — seven-day TVL
  growth is the strongest forward signal in the file, so it weighs more
  than the starter's 0.50.
- **maturity 0.55–1.20** — grows with age up to 4 years; age is how long a
  protocol has run with no token on record (unknown age: 365 days).
- **chains 0.97–1.09** — by chain count; multi-chain spreads risk.
- **category 0.85 / 0.90 / 1.10 / 1.00** — pegged / RWA / bridge 0.85,
  staking / restaking 0.90, fee-earning trading venues (DEX, perps,
  derivatives, basis trading, prediction) 1.10, everything else 1.00.
  A token's upside tracks the fees its protocol can share.

The board covers all 40 watchlist protocols plus every other rankable
pre-token protocol (on this snapshot), with the early
rows chosen by implied value rather than by TVL, so a small fast-growing
protocol is found early. Wrapped, staked and bridged products of tokens
that already trade are left out of the early rows, by the same rule the
field guide uses to build the watchlist. Live-token protocols are set aside and shown at
their market cap, never modelled.

## Farm plans

Each row carries six steps derived from the protocol's own category,
primary chain, TVL tier (capital $50 / $100 / $150 / $250) and age
(hold 30 / 45 / 60 days). Every step states its cost and its time; the
weekly action and the pre-exit points check vary by category (lending,
DEX, perps, bridge, staking, RWA, stablecoin, insurance, and more).

## The dashboard

Dark theme with its own amber/teal palette. Header with the snapshot date
and board counts; KPIs for watchlist coverage, median implied value,
growing rows and live-token count. Controls: search, category filter,
chain filter, TVL range, watchlist-only toggle, live-token tab, and sorting
by rank, implied value, TVL, 7-day change or age. Clicking a row opens a
detail view with the full model breakdown (each multiplier, recomputed in
the page and checked against board.json) and the six farm steps. A
methodology panel states the formula and the disclaimer: implied value is
this entry's own claim, not a price, and it does not predict whether or
when a token is issued.

## Layout

- `main.ts` — entrypoint: reads the snapshot, writes the outputs
- `lib/model.ts` — the radar-2 model, farm plans, snapshot parsing
- `vendor/makefun-board.ts` — make.fun's dossier library (vendored, unchanged)
- `dashboard/` — `index.html`, `app.css`, `app.js` (no network, no external assets)
- `screenshots/` — `1-board.png` (board), `2-detail.png` (detail view),
  `3-filtered.png` (filtered view), each rendered from this dashboard in a
  headless browser

## License

MIT — see `LICENSE`.
