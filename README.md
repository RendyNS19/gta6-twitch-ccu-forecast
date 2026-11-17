[![make.fun](https://make.fun/badge/gta6-twitch-ccu-forecast-01.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01)
[![rank](https://make.fun/badge/gta6-twitch-ccu-forecast-01/RendyNS19/gta6-twitch-ccu-forecast/rank.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01)
[![reward](https://make.fun/badge/gta6-twitch-ccu-forecast-01/RendyNS19/gta6-twitch-ccu-forecast/reward.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01)

# GTA VI Twitch CCU forecast

Forecast, hour by hour, how many people watch Grand Theft Auto VI live on
Twitch over the 24 UTC hours of its release day (2026-11-19), from everything
recorded up to the end of its first hour — five quantiles (5/25/50/75/95%) per
hour, scored by pinball loss on the log scale.

## How it works

`main.ts` (Node 22, stdlib only, no network, fully deterministic) reads every
`*.in` case in `$MAKEFUN_INPUT_DIR` (else `/input`) and writes one
`/output/<id>.out` per case.

Launch-day shape depends strongly on the hour-1 scale: a huge hour-1 print
means the launch is already near its peak, a small one means the day is still
ramping as regions unlock at local midnight. So for each forecast hour the
program fits, over the case's own comparable past launches, a straight line of
`log(viewers_h / p1)` on `log(p1)` and reads the fitted value at this case's
hour-1 print as the median log-ratio. The quantile spread comes from the
fit's residuals: empirical residual quantiles blended 50/50 with a normal
spread (`z_tau * residual sd`, sd floored at 0.22 and capped at 1.2).

Training set: comparables with `Tier: comparable` and a rolling midnight-local
unlock, whose own day-1 peak falls after hour 2 (a peak at hour 1–2 means a
simultaneous global unlock — a different shape), and whose hour-1 print is at
least 5,000 viewers. Filters relax in stages if fewer than 6 launches
qualify; with nothing usable it falls back to the flat house baseline.

## Validation

Practice answers from SullyGnome (the same source the kit used), scored with
the kit's pinball rule:

| case | launch | this forecaster | house baseline |
|------|--------|-----------------|----------------|
| p00 | Marvel's Wolverine | 0.189 | 0.744 |
| p01 | Red Dead Redemption 2 | 0.122 | 0.287 |
| p02 | God of War Ragnarök | 0.115 | 0.215 |
| p03 | Marvel's Spider-Man 2 | 0.215 | 0.303 |
| **mean** | | **0.160** | **0.387** |

Leave-one-out over the nine GTA-like comparables (rolling, comparable, peak
after hour 2): 0.312 vs baseline 0.395.

## Run locally

```sh
mkdir -p out
MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out node main.ts
```

Entered on make.fun · https://make.fun/r/mjwfgnw1
