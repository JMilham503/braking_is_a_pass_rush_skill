# Braking Is a Pass-Rush Skill
NFL Big Data Bowl 2027 — SAL 605 final project

## What you need in data/raw

The analysis uses four files. It does **not** need the game tracking files.

    combine_tracking.csv      the 10 Hz Combine sensor traces  (required)
    player_play_1.csv         per-snap NFL outcomes            (required)
    games.csv                 season and week lookup           (required)
    combine_results.csv       traditional Combine measures     (required)
    players.csv               names, positions, draft info     (required)

The loader matches by filename prefix, so `player_play.csv`,
`player_play_1.csv` or several split parts all resolve without edits. Put the
files in `data/raw` or change `DATA_DIR` at the top of the .Rmd.

The ~2.3 GB of game tracking is not read. The NFL-side metrics we use
(pressure, get-off, time to pressure) are already computed per snap in
`player_play`, which is itself derived from that tracking. See "Possible
extension" below if you want to use the raw frames directly.

## Packages

    install.packages(c("data.table", "ggplot2", "glmnet", "cluster"))

`cluster` ships with R. Savitzky-Golay smoothing is implemented inline with
hardcoded coefficients, so neither `signal` nor `pracma` is required.

## Running it

    rmarkdown::render("bdb2027_analysis.Rmd")

First run is the slow one: it reads ~80 MB of tracking, derives metrics for
roughly 1,900 drill traces, then fits the cross-validated models. The heavy
chunks are cached, so re-knits are fast. Expect several minutes initially.

If a chunk errors, the cache can go stale. Delete the
`bdb2027_analysis_cache/` folder and re-knit.

## Reading the output

Every number in the prose is computed inline by the code, not pasted in. What
the document says is whatever your run produced. Reference values from the
knit of October 12 (R, glmnet):

    DL get-off, traditional measures        CV R2  +0.444
    DL get-off, sensor metrics              CV R2  +0.110
    DL get-off, traditional + sensor        CV R2  +0.472
    DL pressure above expected, traditional CV R2  +0.015
    DL pressure above expected, sensor      CV R2  +0.146
    DL pressure above expected, both        CV R2  +0.179
    OL pressure above expected, traditional CV R2  -0.061
    OL pressure above expected, sensor      CV R2  -0.074

    archetype weight difference             ANOVA p = 0.82
    archetype pressure difference           ANOVA p = 0.033
    archetype added to weight + forty       F test p = 0.77 (51 players with a forty)
    braking, controlling for mean speed     t = 3.36, p = 0.0013

Not reproduced in the Rmd: the permutation test (p = 0.0033), leave-one-out
check, and bootstrap interval on the sensor minus traditional gap
(90% CI -0.251 to +0.518) come from the Python validation on the same data and
model specification. The Limitations section states their conclusions.

If a rerun moves any of these by more than about 0.05, something has changed
and is worth a look.

## Course requirements

    Research questions          three, stated explicitly in the first section
    Advanced topic              two of the four: penalized regression and
                                k-means clustering with silhouette selection
    Visualizations              eight (requirement is three), plus a standalone animation for the recording
    Everything in R             yes, single .Rmd, code appendix at the end
    Recording                   see PRESENTATION_SCRIPT.md

## Contest requirements

About 1,900 words in the body and eight figures, within the 2,000-word and 10-figure limits.
Code is in the appendix rather than the body. The analysis uses player tracking
data, which is the hard gate for being scored at all.

## Possible extension, if you get time

The strongest addition would use the game tracking you already have: compute
each defensive lineman's actual in-game deceleration from the NFL frames, and
test whether Combine braking predicts in-game braking. That turns the argument
from "Combine movement predicts an outcome" into "Combine movement predicts the
same movement on Sundays, and that movement produces pressure," which is a
direct Combine-to-NFL link rather than an inferred one.

It means processing 22.5 million frames, which `data.table` handles but not
quickly. Worth doing only if the deadline allows.
