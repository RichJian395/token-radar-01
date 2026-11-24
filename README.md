# Token Radar — pre-token protocols ranked by implied value

[![Token Radar](https://make.fun/badge/token-radar-01.svg)](https://make.fun/c/token-radar-01)
[![Rank](https://make.fun/badge/token-radar-01/RichJian395/token-radar-01/rank.svg)](https://make.fun/c/token-radar-01)
[![Reward](https://make.fun/badge/token-radar-01/RichJian395/token-radar-01/reward.svg)](https://make.fun/c/token-radar-01)

A token launch radar for [make.fun's token-radar-01 bounty](https://make.fun/c/token-radar-01):
it reads a sealed snapshot of 758 DefiLlama protocols, ranks every pre-token
protocol by the implied value its token would carry, and shows a reader how to
farm each one. The dashboard is a static page with no network calls.

## What it answers

- **Which protocols have no token on record, and what would a token be worth?**
  The board covers the snapshot's whole 40-protocol watchlist (TVL ≥ $50M, no
  token on record) and adds every other rankable pre-token protocol below the
  floor — 198 rows in all — ranked by implied value.
- **What should a reader do about it?** Every row carries a seven-step farm
  plan with a cost and a time for each step, derived from the protocol's own
  category, chains, TVL and age.
- **What is already trading, or wrapped?** Live-token protocols are set aside
  in their own view with their market caps, and wrapped, staked and bridged
  products of tokens that already trade (WBTC, Binance staked ETH) are set
  aside with the reason — never mixed into the ranking. A wrapped token is
  not a launch.

"Records no token" is what the snapshot says, never proof a protocol has
none. Nothing here predicts whether or when a protocol launches a token.

## The model (radar-2)

Implied value follows only from the snapshot's own figures, through four
multipliers. No protocol name is special-cased; the same function scores
every row.

```
implied = round( tvl_usd x category_multiple
                 x (1 + clamp(change_7d, -50, +100) / 100 x 0.5)
                 x (0.6 + min(age_days, 1095) / 1095 x 0.4)
                 x (1 + 0.02 x min(chains, 5)) )
```

| Multiplier | Value | Why |
| --- | --- | --- |
| Category multiple | fee engine ×0.30, neutral ×0.22, pass-through ×0.12 | TVL is the economy a token would tax; the multiple is the share of it a token can plausibly capture. Fee engines (Dexs, lending, derivatives, yield, CDP, allocators) earn fees on every dollar that passes through. Pass-through infrastructure (bridges, chains, liquid staking) mostly moves value that belongs to another asset. |
| Growth | 7d change clamped to −50%…+100%, weight 0.5 | Momentum. The clamp matters: the snapshot's largest 7d move is over +4000%, and one wild week must not crown a protocol. |
| Maturity | 0.6 → 1.0 over 3 years | How long the protocol has run with no token on record. The longer it has run, the more value has accrued for a launch to capture. Unknown listing date uses 365 days. |
| Chain breadth | +2% per chain, up to 5 | Each extra chain is another user base and another place the token would be usable. |

The dashboard's model panel states the formula and its constants, and its
sandbox re-ranks the board live when you move the growth weight, the maturity
floor or the chain bonus. The submitted `board.json` always uses the published
constants above.

## The farm plan

Seven steps per protocol, each with a cost and a time:

1. Fresh wallet, seed kept offline — under $3 of gas, 5 minutes.
2. The main position, chosen by category (supply, provide liquidity, bridge,
   stake, margin, vault, mint) — $50–$250 of capital by TVL, 15 minutes.
3. One qualifying action a week — about $2 of gas a week, 10 minutes a week.
4. Hold at least 30–60 days (longer for younger protocols) so the activity is
   counted in more than one snapshot.
5. Check the protocol's own points or rewards page before withdrawing; never
   pay to claim.
6. Watch for a token announcement only on the protocol's own site and official
   account; ignore airdrop messages and claim links sent to you.
7. Keep a farm log of the wallet, dates and transactions.

## Running it

Node 22.18 or later, standard library only — nothing to install.

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <path to snapshot.json>
```

This writes `out/board.json` (the ranked board), `out/data.js` (the
dashboard's data), `out/dashboard/` (the page), `out/screenshots/` (the three
screenshots) and `out/dossier.json` (the judge's summary, built with the
vendored `vendor/makefun-board.ts`). Open `out/dashboard/index.html` in a
browser; it renders from `data.js` with no network.

The program runs twice in grading and both runs must write the same bytes:
the ranking is fully deterministic (ties break by slug).

## Layout

```
main.ts                  entrypoint: reads the snapshot, writes /output
lib/model.ts             the radar-2 model, the board, the farm plans
vendor/makefun-board.ts  make.fun's dossier library (vendored, unchanged)
dashboard/               index.html, app.css, app.js — the static page
screenshots/             1-board.png, 2-detail.png, 3-filtered.png
```

## Data

The snapshot is DefiLlama's public protocol index as fetched once by the
bounty house (`as_of` 2026-11-14, floor $50,000,000, 758 protocols, 40 on the
watchlist). It is sealed input: the entry reads it, never updates it.
