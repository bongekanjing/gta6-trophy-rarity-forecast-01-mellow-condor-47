# gta6-trophy-rarity-forecast-01-mellow-condor-47

[![make.fun · gta6-trophy-rarity-forecast-01](https://make.fun/badge/gta6-trophy-rarity-forecast-01.svg)](https://make.fun/bounty/gta6-trophy-rarity-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-trophy-rarity-forecast-01)
[![rank](https://make.fun/badge/gta6-trophy-rarity-forecast-01/bongekanjing/gta6-trophy-rarity-forecast-01-mellow-condor-47/rank.svg)](https://make.fun/bounty/gta6-trophy-rarity-forecast-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=gta6-trophy-rarity-forecast-01)
[![reward](https://make.fun/badge/gta6-trophy-rarity-forecast-01/bongekanjing/gta6-trophy-rarity-forecast-01-mellow-condor-47/reward.svg)](https://make.fun/bounty/gta6-trophy-rarity-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-trophy-rarity-forecast-01)

Forecasts, for each of Grand Theft Auto VI's 55 PlayStation 5 trophies, the earn
rate Exophase will print at 2026-11-26T00:00:00Z, as five quantiles per trophy.

## The idea

Exophase prints each trophy's share of the profiles it tracks. That share is a
ratio of two counts, and both have to be forecast:

```
rate_i = 100 * C_i / N
```

`N` is the players tracked; `C_i` is how many of them hold trophy `i`. The rate
moves because the two grow at different speeds, and measured on the real page the
difference is large:

| | launch night | now (23 h later) | growth |
|---|---|---|---|
| players tracked `N` | 638 | 20,000 | x31 |
| chapter 1 earners | 8 | 146 | x18 |
| the platinum's earners | 4 | 6 | x1.5 |
| chapter 1 rate | 1.25% | 0.73% | x0.58 |
| the platinum's rate | 0.63% | 0.03% | x0.05 |

Writing the count as a power of the population, `C_i = C_i0 (N/N0)^gamma_i`, gives

```
rate_i = rate_i0 * (N/N0)^(gamma_i - 1)
```

so each trophy needs one number, its elasticity `gamma_i`, plus a forecast of `N`.
`gamma = 1` is a trophy every late joiner earns and whose rate therefore holds;
`gamma` near 0 is a completionist trophy whose count is frozen while the crowd
doubles, so its rate is diluted nearly one-for-one. The story trophies sit
between. That is why the platinum falls 20-fold while the tutorial stays at 100%,
and it is the structure the practice cases alone cannot show.

Each `gamma_i` is a regression of log count on log players tracked through every
refresh the case's history holds, not a ratio of its first and last read. Launch
night's reads hold four earners for the platinum: one more or one fewer moves that
ratio by a quarter, so the end-to-end ratio lets noise set the exponent. The
regression also yields its own residual, which is what the band uses for the
exponent's error rather than a constant.

## The refresh

The page is a step function. Exophase republishes its counts on its own schedule,
so between refreshes every rate is frozen and `N` never moves. The rules describe
the first refresh: the page read unchanged at 638 players from launch night to
04:00, then showed 3,868 at 04:30 — with every rate but the 100% one *lower*.
Counts held across it; only the denominator moved.

Both practice cases sit entirely before that refresh and ask for the rates after
it, which is exactly why a persistence forecast fails them: a flat history means
the page is stale and a refresh is due, not that nothing will change. The model
reads a stale page as about to be diluted, and splits the population growth into
the catch-up such a refresh publishes (counts held, `gamma = 0`) and the ordinary
progression that follows (each trophy's own `gamma`).

## The spread

The score is a pinball loss on the logit scale, so the width of the band is scored
as strictly as its centre. Three uncertainties enter it: the elasticity's own
error, how far `N` runs, and where a refresh lands. On the 90-minute practice
horizon the first two are small and the refresh is everything; on the week-long
final the forecast of `N` dominates.

Sized against the one honest out-of-sample test available before the final: the
launch-night state and the 04:30 refresh as history, forecasting the live page 19
hours later, scored against rates the page actually printed. On that test the
model scores 0.405 against persistence's 0.708, with 90% coverage of 0.927.

## Run it

```sh
MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out python3 main.py
```

Reads every `*.in` case in the input directory in name order and writes one
`<id>.out` for each. Standard library only, no network, no randomness: the same
input gives byte-identical output, and a case takes about 5 ms.

## Licence

MIT.
