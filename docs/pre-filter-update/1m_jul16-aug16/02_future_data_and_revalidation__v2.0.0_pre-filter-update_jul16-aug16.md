# Future Data Collection & Re-Validation Plan

Status: the conclusions in
`01_baseline_forecast_model__v2.0.0_pre-filter-update_jul16-aug16.md` and
`03_feature_extraction_reference__v2.0.0_pre-filter-update_jul16-aug16.md`
are provisional, based on ~30-63 days of `pipeline_version=2.0.0` data.
This document defines what counts as enough data, the checks to be re-run,
and the pass/fail condition for each.

Sources: all figures are from the first month of pipeline 2.0.0 data
(2026-07-16 to 2026-08-16). No notebook output saved in this repository
reproduces them.

## Limits of the 30-63 day window

- Two half-window checks (day 1-15 vs day 16-31) found that oil's std
  nearly doubled between halves instead of stabilizing (0.0982 -> 0.1826)
  and that pulses' nonzero-day rate rose from 50% to 100%. A single
  full-window summary shows neither.
- `price_lag_14` / `rolling_mean_14` are structurally NaN for most
  products at this depth (median obs/product 8-27, well under 14).
- Only ~4-9 weekly cycles exist depending on window definition, too few to
  test `day_of_week`/`is_weekend`. The day/night (session-timing) question
  was checked separately and showed no effect.
- The per-product eligibility gate (nonzero CV + recency filter) has been
  run once, at one depth; its stability has not been checked the way the
  category-aggregate std was.

## Milestone: ~60 days on `pipeline_version=2.0.0` only

"60 days" is counted as calendar days since the first
`pipeline_version=2.0.0` row, **not** since the refactor was deployed
(deploy date and first-clean-row date may differ) and **not** counting
backfilled `1.0.0` history.

## Checks defined for the 60-day checkpoint

### 1. Per-product eligibility gate (per category)
The same two-step gate (CV > 0, then last observation within 3 days of
window end) is applied independently per category and compared against the
30-day results:

| Category | 30-day eligible | 30-day eligible % |
|---|---|---|
| sugar | 47 / 133 | 35.3% |
| ghee | 169 / 561 | 30.1% |
| tea | 57 / 206 | 27.7% |
| rice | 174 / 671 | 25.9% |

**Pass condition:** eligible % stays within ~5-10 points of the 30-day
number for each category, and the set of eligible product IDs overlaps
substantially with the 30-day set (not a different rotating cast of
products). Large swings in either direction indicate the gate was
measuring noise rather than a property of the category.

### 2. Category-aggregate volatility (std / nonzero-day rate)
`cat_price_change_pct` describe() and the groupby std/nonzero table are
recomputed as a 3-4 way split (not two halves) to distinguish trending
from bouncing:

| Category | full-31d std | H1 std | H2 std | 30-day verdict |
|---|---|---|---|---|
| oil | 0.1455 | 0.0982 | 0.1826 | disqualified: got worse, not better |
| sugar | 0.0654 | 0.0716 | 0.0612 | stable |
| flour | 0.0422 | 0.0363 | 0.0473 | moderate drift |
| dairy | 0.0386 | 0.0266 | 0.0482 | moderate drift |
| tea | 0.0304 | 0.0268 | 0.0339 | reasonably stable |
| pulses | 0.0257 | 0.0256 | 0.0261 | std stable, but nonzero% rose 50%->100%; cause not established |
| ghee | 0.0196 | 0.0192 | 0.0200 | most stable |
| rice | 0.0135 | 0.0055 | 0.0181 | grew 3.3x; not established whether real or still settling |

**Pass condition:** std and nonzero% stay in a consistent band across all
splits, not just two. A category whose std is still moving monotonically in
one direction at 60 days (as oil's did across 31) does not pass, regardless
of which direction it moves.

### 3. Oil: minimum-sample floor on the daily median
Oil's instability is attributed to thin daily samples: too few
observations per day, causing composition-driven jumps in the daily median
rather than real price movement. The evidence is single-day swings of
-33.7% to +55.9% in `cat_price_change_pct`. The check defined for oil:

- A `MIN_SAMPLE_FOR_DAILY_MEDIAN` floor (matching the scraper's
  `MIN_SAMPLE_FOR_MEDIAN=10` from `validation.py`) is applied to the
  category-daily aggregation step. Days with fewer valid oil observations
  than the floor are set to `NaN` or forward-filled instead of being
  computed from a thin sample.
- The std/nonzero check is re-run. If std falls to a range comparable to
  sugar/dairy, oil is eligible for inclusion. If it does not, the
  volatility is structural to the category rather than a sampling
  artifact, and oil stays excluded.

### 4. Pulses: nonzero-day rate
The check compares `n_days` and valid-observation counts for pulses between
the first and second half of the window. If pulses had materially fewer
valid daily observations in H1, the 50% nonzero rate in H1 reflects days
with too few products to move rather than lower volatility. If observation
counts are comparable across both halves, the jump is a behavioral signal
in its own right.

### 5. Lag/rolling feature coverage
The % of product-days where `price_lag_14` / `cat_rolling_mean_14` are
non-NaN is recomputed. At 60 days it is expected to be populated for
roughly half the window for most products. If it is still mostly NaN,
history is not deep enough for these features.

### 6. Weekly seasonality
At ~60 days (~8-9 weeks), `day_of_week`/`is_weekend` can first be tested
(v1.0.0's rule: >=2 full weekly cycles required). The test uses the same
category-median and paired-test approach as the day/night (session-timing)
check, with day-of-week in place of morning-vs-evening.

### 7. Baseline model re-evaluation
The model in `01_baseline_forecast_model__v2.0.0_pre-filter-update_jul16-aug16.md`
and its three baselines (naive persistence, 7-day rolling mean,
discount-aware naive) are re-run on the 60-day window, and MAE/RMSE are
compared with the 30-day run. A win is the feature model's advantage over
the rolling-mean baseline holding steady or improving, not only a lower
absolute error, since more data lowers error regardless of model quality.

## Criterion for committing to category selections

The criterion is **consistency across at least two non-overlapping
windows**, not a single good number. Re-running the checks above at day 60
(or as a 3-way split within a 60-day window) should not contradict the
day-30 conclusions. Categories and features whose conclusions flip between
windows (as oil and pulses did between the two 15-day halves) are not
committed, regardless of the most recent check.

## Scope at this checkpoint

- Per-product forecasts are not produced.
- Oil and pulses are not added to any model before checks 3 and 4 are
  complete.
- `day_of_week`, `week` and `month` are not used before check 6.
- A single new data pull does not by itself change a category selection;
  changes are made by comparing against the prior window's figures in this
  document.