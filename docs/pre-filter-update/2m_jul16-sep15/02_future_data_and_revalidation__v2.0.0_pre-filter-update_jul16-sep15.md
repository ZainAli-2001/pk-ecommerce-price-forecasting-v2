# Future Data Collection & Re-Validation Plan

Status: **60-day checkpoint complete** (61-day window, 2026-07-16 to
2026-09-15, `pipeline_version=2.0.0` only). Most of the checks defined
at the 30-day milestone have been re-run; results are compared against the
30-day baseline throughout. Open items are listed at the end.

Sources:

- Backed by saved notebook outputs: the full-window std values
  (`notebooks/pre-filter-update/feature_engineering_gated.ipynb`, cell 20),
  the 88,611 daily rows (cell 1) and the eligible total across ghee, rice and
  tea (464 of 1,554 products, cell 7).
- Computed from the `forecast_data` CSVs written by the gated notebook
  (`category_daily_aggregate_features.csv`,
  `selected_categories_per_product_features.csv`,
  `daraz_daily_with_discount_features.csv`; not included in this repository):
  per-category eligible counts, lag/rolling coverage, the three-window
  volatility table, the pulses window table, the flour outlier check, and the
  spike split for ghee, rice and tea.
- From `forecasting_stage2.ipynb` runs on the 30-day and 60-day data, whose
  outputs are not saved: session-grain panel statistics, paired product-days
  and p-values, spike totals and the span ratio. The saved stage-2 run
  belongs to the later commodity filter and does not reproduce these.
- Baseline results: `results/pre-filter-update/cp2_2m_jul16-sep15/baseline/`.

## Month-1 findings

Two 15-day half-window checks at 30 days showed oil's std nearly doubling
instead of stabilizing and pulses' nonzero-day rate rising from 50% to
100%; a single full-window summary shows neither. `price_lag_14` and
`rolling_mean_14` were structurally NaN for most products. Only ~4-9 weekly
cycles existed, too few for `day_of_week` testing.

## Panel stability and churn

Session-grain (pre-collapse) comparison:

| Metric | 30-day | 60-day | Read |
|---|---|---|---|
| Daraz products | 2,891 | 3,139 (+8.6%) | Small growth |
| Daraz rows | 81,569 | 158,002 (+93.7%) | Tracks elapsed time |
| Daraz obs/product (median) | 24 | 36 | **1.5x — real accumulation** |
| Daraz flat-price % | 55.6% | 50.9% | Improving |
| Naheed products | 152 | 153 | Flat |
| Naheed obs/product (median) | 56 | 102 | 1.8x growth |

Product count barely grew while obs/product grew 1.5-1.8x: a stable core
panel accumulating history, not a churning catalog diluting the panel with
short-history products. A separately reported "5.5k→7.8k" product count
refers to the `products.csv` master catalog (all products ever seen, subject
to normal Daraz listing turnover), not the modeling panel; the two counts
answer different questions.

## Per-product eligibility gate (ghee, rice, tea)

| Category | 30-day eligible | 60-day eligible | Change |
|---|---|---|---|
| ghee | 169 / 561 (30.1%) | 189 | +20 (+11.8%) |
| rice | 174 / 671 (25.9%) | 197 | +23 (+13.2%) |
| tea | 57 / 206 (27.7%) | 78 | +21 (**+36.8%**) |

Sugar, oil, flour, dairy, pulses were not re-gated at 60 days; the gated
notebook's `SELECTED_CATEGORIES` output covers only ghee/rice/tea.
Obs/product within the eligible sets: ghee median 54 (range 4-62), rice
median 46 (range 3-62), tea median 56.5 (range 2-62), past the 14-day lag
threshold for the median product in each.

All three eligible sets grew rather than shrinking or churning, consistent
with the panel-stability result above. Tea has the largest proportional
growth (+36.8%); the feature reference's per-product eligibility section
gives tea's rank history.

## Lag and rolling feature coverage (ghee, rice, tea)

| Category | `price_lag_14` non-null | `rolling_mean_14` non-null |
|---|---|---|
| ghee | 71.4% | 100% |
| rice | 67.8% | 100% |
| tea | 70.9% | 100% |

At 30 days these were structurally NaN for most products; now 2/3-3/4 of
rows have real 14-day lags. Coverage was not checked for
oil/sugar/flour/dairy/pulses. Their lower `median_obs` figures (feature
reference, daily-grain table) suggest it would still be thin, but this has
not been verified.

## Category-aggregate volatility

Full 61-day window plus a 3-way ~20-day split (W1: Jul16-Aug5, W2:
Aug6-25, W3: Aug26-Sep15):

| Category | Full std | W1 | W2 | W3 | Trend |
|---|---|---|---|---|---|
| **oil** | 0.1484 | 0.0957 | 0.1611 | 0.1807 | **Monotonic climb** |
| **flour** | 0.1299 | 0.0333 | 0.0609 | **0.2138** | **Same pattern as oil; outlier check below** |
| sugar | 0.0589 | 0.0755 | 0.0428 | 0.0559 | Bouncing, no trend |
| dairy | 0.0498 | 0.0256 | 0.0519 | 0.0653 | Mild upward drift |
| tea | 0.0279 | 0.0250 | 0.0369 | 0.0204 | Bouncing, no trend |
| pulses | 0.0275 | 0.0245 | 0.0307 | 0.0282 | Stable |
| ghee | 0.0171 | 0.0170 | 0.0211 | 0.0130 | Stable, low |
| rice | 0.0153 | 0.0087 | 0.0117 | 0.0222 | Mild upward drift, still lowest-ish |

**Pass condition:** std/nonzero% land in a consistent band across all
three windows, not trending monotonically in one direction.

**Result:** ghee, rice, tea, pulses, sugar and dairy pass (stable or
bouncing without a clear trend). Oil and flour fail: both show a clear,
worsening trend across all three windows rather than settling.

### Flour outlier check

`.describe()` on flour's full-window `cat_price_change_pct`: min
-28.6%, max +59.8%, implausible as genuine single-day retail price
moves. W3 alone contains eight separate days with >19% absolute moves.
Flour has the smallest product count of any category (133-150
throughout). The pattern matches oil's and is attributed to the same
cause: thin daily samples causing compositional swings in the category
median rather than real price movement.

## Oil

Four metrics show instability:

1. Category-aggregate std: 0.1484 full-window, monotonic climb across
   all three windows (0.0957→0.1611→0.1807)
2. Paired-Wilcoxon std: 6.179 at 60 days vs 1.013 at 30 days (6.1x),
   while every other category stayed roughly flat
3. Span-ratio: 25.2 at 60 days vs 18.2 at 30 days (+7.0, largest
   movement of any category)
4. `.describe()` tails: -33.7% to +55.9% single-day swings

Condition for reconsidering oil: a `MIN_SAMPLE_FOR_DAILY_MEDIAN` floor
(matching the scraper's existing `MIN_SAMPLE_FOR_MEDIAN=10`) applied to the
category-daily aggregation step, excluding or forward-filling days with too
few valid observations, followed by a re-run of the four checks above.

## Flour

Same diagnosis and same condition for reconsideration as oil (see above).
Flour was not flagged at 30 days; it became visible once the 3-way
volatility split was run.

## Pulses nonzero-day rate

| Window | n_days | pct_nonzero |
|---|---|---|
| W1 (Jul16-Aug5) | 20 | 61.9% |
| W2 (Aug6-25) | 20 | 100.0% |
| W3 (Aug26-Sep15) | 21 | 95.2% |

`n_days` is stable (~20/window), so W1 has no coverage gap, which rules out
missing data as the cause of the 30-day 50%→100% jump. Pulses had a
quieter early period followed by a sustained increase in day-to-day
movement from W2 onward, and the higher level held through W3 rather than
reverting. The jump is a behavioral shift, not a coverage artifact.
Pulses is not disqualified on this metric. Its flat-rate (68.4% at 60
days, worst alongside dairy) and the absence of a re-run eligibility gate
remain reasons for caution before including it in any model.

## Day/night (session-timing) effect

Median gap check (60-day): 6 of 8 categories match their 30-day result
(dairy/flour/oil/ghee at or moving toward 0.00%, rice/tea small and in the
same direction). Sugar diverged (-0.33% → -5.13%).

Paired-Wilcoxon (60-day, n=68,142 paired product-days, up from 33,947):
7 of 8 categories show p<0.05 (4 of 8 at 30 days). This is the expected
result of increased statistical power finding smaller effects, not an
effect that became real. **Effect sizes are negligible in every category:**
the median paired difference is exactly 0.0 everywhere, and means are
~0.0001-0.02% of typical price levels. Sugar's paired mean (-0.124) is the
largest in absolute terms of the eight categories but is ~0.02% of its
price level (~600-700 PKR/kg), so it does not account for a -5.13% median
gap. The sugar median-gap divergence is therefore consistent with a
composition artifact (aggregate-level, sensitive to which products are
present) rather than a same-product day/night price effect (paired-level,
not composition-sensitive).

Conclusion: `hour`/session timing is not used as a feature. The conclusion
rests on effect size, not p-values: with more data nearly every category
crosses p<0.05 while the effect stays negligible.

## Spike detection

| Metric | 30-day | 60-day |
|---|---|---|
| Daily rows | 47,622 | 88,611 |
| Spikes | 556 (1.17%) | 1,194 (1.35%) |
| Discount-driven | 182 (32.7%) | 411 (34.4%) |

The spike rate rose modestly (1.17% to 1.35%) after holding flat between
v1.0.0 and the 30-day v2.0.0 data. The discount-attributed share rose from
32.7% to 34.4% after falling from v1.0.0's 44.1%; three points are not
enough to establish a trend. Within the gated ghee/rice/tea categories:
473 spikes (188/181/104), 34.0% discount-driven, in line with the
dataset-wide rate.

## Baseline model results (see `01_baseline_forecast_model__v2.0.0_pre-filter-update_jul16-sep15.md`)

Improvement vs rolling7: ghee 42.4%, rice 31.4%, tea 21.9%; all three
also beat naive persistence. Sugar (2.6% vs rolling7) fails against naive
persistence, which is why the pass condition requires beating naive.

## Open items

1. The `MIN_SAMPLE_FOR_DAILY_MEDIAN` floor has not been applied to oil and
   flour, and the volatility and spread checks have not been re-run with
   one. Neither category is in any model.
2. The per-product eligibility gate has been re-run at 60 days only for
   ghee, rice and tea. Sugar's 30-day eligible rate (35.3%) was the best of
   any category; whether it held is not known, and sugar failed the
   aggregate-model check.
3. Rating/review correlation with price movement has not been tested.
4. Dairy's `median_obs` is 11.0 at both 30 and 60 days (session-grain daily
   table), the only category with no growth while every other category grew;
   the cause is not established.
5. The discount-attributed share of spikes moved 44.1% → 32.7% → 34.4%;
   whether the recovery is real is not established.

## Pending items

Items 1-2 depend on the sample-floor change and on data already collected,
not on additional calendar time. Items 3-5 depend on further data volume
(for example 90-120 days).