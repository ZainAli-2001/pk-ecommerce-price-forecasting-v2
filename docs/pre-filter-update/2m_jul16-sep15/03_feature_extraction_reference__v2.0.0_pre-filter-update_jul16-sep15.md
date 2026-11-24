# Feature Extraction Reference — What, How, Why, Status

Status: updated at the 60-day checkpoint (61-day window, 2026-07-16 to
2026-09-15, `pipeline_version=2.0.0`). Where a status differs from the
30-day version of this document, both figures are shown, because several
categories changed conclusions between checkpoints.

Sources:

- Backed by saved notebook outputs
  (`notebooks/pre-filter-update/feature_engineering_gated.ipynb`): the
  60-day per-category daily-grain table (cell 3), the full-window std values
  (cell 20), the eligible total across ghee, rice and tea (464 of 1,554
  products, cell 7), and `SELECTED_CATEGORIES` (cell 5). The 30-day values in
  the per-category table are not reproduced by saved outputs.
- Computed from the `forecast_data` CSVs (not included in this repository):
  per-category eligible counts, lag/rolling coverage, the three-window trend
  and the spike split for ghee, rice and tea.
- From `forecasting_stage2.ipynb` runs whose outputs are not saved:
  session-grain figures, the day/night checks and the Naheed figures.
- Baseline results: `results/pre-filter-update/cp2_2m_jul16-sep15/baseline/`.

## Data sources and grain

- `product_prices` — append-only time series, `pipeline_version` column
  (`1.0.0` backfilled, `2.0.0` for new rows).
- `products` — deduped by `url`. A separately reported "5,522 → 7,800"
  product count refers to the `products.csv` **master catalog** (all
  products ever seen, subject to normal Daraz listing churn), a different,
  larger number than the modeling-panel product counts used in this document.
- Session grain: ~2 scrapes/day (6AM/6PM PKT). Daily grain: sessions
  collapsed into one row per product-day (no day/night effect found at
  either checkpoint; see Calendar features).
- Daraz only. Naheed is captured but excluded from forecasting (see
  Cross-source).

## Target

`unit_price` (PKR/kg or PKR/liter). Models are built per `unit_type`; kg
and liter series are not mixed.

## Identifiers (not features)

`url`, `name`, `category`, `source`, `unit_type`, `unit_value`,
`run_id`, `scraped_at`/`obs_date`, `scraper_version`,
`pipeline_version`. `category`/`unit_type` may be used as categorical
splits.

## Calendar features

| Feature | Status | Evidence |
|---|---|---|
| `hour` / day-vs-night | **Not used: no effect found at either checkpoint** | 30-day: 3-part check (median gaps ~0%, Wilcoxon p>0.05 in every category, discount-conditioned coin-flip). 60-day re-run (n=68,142 paired product-days, up from 33,947): 7/8 categories show p<0.05, but **every effect size is negligible**: median paired diff is 0.0 in all 8 categories, means ~0.0001-0.02% of price levels. More significant p-values with more data is expected statistical power, not an emerging effect. Sugar's 60-day median-gap divergence (-0.33%→-5.13%) does not show up in the paired-difference check (sugar's paired mean, -0.124, is ~0.02% of its price level), consistent with a composition artifact rather than a same-product effect. The conclusion rests on effect size, not p-value. |
| `day_of_week`, `is_weekend` | Untested | ~8-9 weekly cycles at 60 days (~4-9 at 30 days), still short of a confident test |
| `week`, `month` | Not usable: no variance | Needs multi-month history |

## Lag features

| Feature | Grain | Status |
|---|---|---|
| `price_lag_1`, `price_lag_7` | daily | Usable |
| `price_lag_2` | daily | Usable. The category-aggregate column `cat_price_lag_2` is present in `category_daily_baseline_features.csv`; its presence in the gated per-product feature output is not confirmed |
| `price_lag_14` | daily | **Usable for ghee/rice/tea** (provisional at 30 days, when it was NaN for the majority of products). 60-day, gated categories: ghee 71.4%, rice 67.8%, tea 70.9% non-null. Not verified for oil/sugar/flour/dairy/pulses; their lower `median_obs` (daily-grain table below) makes similar coverage unlikely but unconfirmed. |

## Rolling statistics

| Feature | Status |
|---|---|
| `rolling_mean_7`/`min_7`/`max_7` | Usable once obs count approaches the window |
| `rolling_mean_14` | **Usable for ghee/rice/tea**: 100% non-null in the gated per-product file at 60 days. Not verified elsewhere. |
| `rolling_std_7` | Same caveat as at 30 days: not relied on below ~4-5 observations |

## Momentum features

| Feature | Status |
|---|---|
| `price_change`, `price_change_pct` | Usable, read together with discount state |
| `price_accel` | Not computed in the gated pipeline |

## Discount features — still the strongest validated signal

| Feature | Status | Evidence |
|---|---|---|
| `is_discounted`, `discount_pct`, `discount_rate` | **Primary feature, high confidence** | `no_discount->discount`: 100% drop by definition. `discount->discount`: deepens (~89.7% v1.0.0, consistently negative in v2.0.0). Confirmed across two pipeline versions and three sample-size checkpoints (v1.0.0 ~14k rows, v2.0.0 30-day ~48k rows, v2.0.0 60-day ~89k rows). |
| `has_discount`, `rolling_discount_rate` | Usable | Used in the baseline model; the discount-aware baseline nearly matched the full model for tea and sugar, suggesting discount features carry most of the forecastable signal in those two categories |

## Rating / review features

| Feature | Status |
|---|---|
| `has_rating`, `has_review` | Usable as auxiliary features, coverage stable (~49-62% depending on version/source) |
| `rating_clean`, `review_count_clean`, `log_review_count`, `rating_weighted` | Untested for price correlation; computed and available, exploratory only |

## Spike detection

| Feature | Status |
|---|---|
| `is_spike` | Label, not input feature. The rate rose from 1.17% to 1.35% (30-day→60-day) after holding flat from v1.0.0 to the 30-day data. The discount-attribution share moved 44.1% (v1.0.0) → 32.7% (30-day) → 34.4% (60-day); whether the recovery is noise or real is not established. Within gated ghee/rice/tea: 473 spikes, 34.0% discount-driven, in line with the dataset-wide rate. |

## Cross-source (Naheed)

`price_gap`/`price_gap_pct`: dropped. Coverage remained ~20% at 60 days
(consistent with both prior checkpoints), indicating a structural
scrape-timing mismatch rather than a measurement fluke. Naheed also has
56.9% flat-price at 60 days (71.1% at 30 days, improving like Daraz but
still weak) and 0% rating/review coverage throughout.

## Per-category daily-grain table (flat-rate / CV / obs)

| Category | n_products (30d→60d) | pct_flat (30d→60d) | median_obs (30d→60d) |
|---|---|---|---|
| sugar | 133→151 | 57.9→55.6 | 25.0→38.0 |
| rice | 671→708 | 61.0→55.8 | 13.0→18.0 |
| tea | 206→242 | 62.6→55.8 | 14.0→17.0 |
| ghee | 561→604 | 61.7→57.6 | 14.0→24.0 |
| oil | 531→595 | 63.8→59.3 | 8.0→12.0 |
| pulses | 495→512 | 77.2→68.4 | 27.0→51.0 |
| flour | 133→150 | 71.4→69.3 | 13.0→17.5 |
| dairy | 161→177 | 74.5→70.6 | 11.0→11.0 (unchanged; see open items) |

All categories improved on flat-rate. Rankings compressed at the top
(sugar/rice/tea within 0.2 points of each other at 60 days), so these
categories are comparably tradeable rather than strictly ranked.

## Category-aggregate volatility (std of `cat_price_change_pct`)

| Category | Full 61-day std | 3-way trend | Verdict |
|---|---|---|---|
| oil | 0.1484 | monotonic climb (0.0957→0.1611→0.1807) | **Excluded: instability on 4 independent metrics** |
| flour | 0.1299 | monotonic climb (0.0333→0.0609→0.2138) | **Excluded: same pattern as oil (`.describe()` outlier check)** |
| sugar | 0.0589 | bouncing, no trend | Usable for volatility, but **failed the baseline model check** (see below) |
| dairy | 0.0498 | mild upward drift | Usable |
| tea | 0.0279 | bouncing, no trend | Usable |
| pulses | 0.0275 | stable | Usable; nonzero-rate question explained (behavioral shift, see revalidation document) |
| ghee | 0.0171 | stable, low | **Most trustworthy** |
| rice | 0.0153 | mild upward drift, still lowest-ish | **Most trustworthy** |

Oil and flour are held until a `MIN_SAMPLE_FOR_DAILY_MEDIAN` floor
(mirroring `validation.py`'s existing `MIN_SAMPLE_FOR_MEDIAN=10`
pattern) is applied to the daily aggregation step. Both show implausible
single-day swings (oil: -33.7% to +55.9%; flour: -28.6% to +59.8%),
attributed to thin daily samples distorting the median rather than real
price movement.

## Per-product eligibility gate

Two-step filter (CV>0, then last-obs within 3 days of window end).
`SELECTED_CATEGORIES` per the gated notebook = **ghee, rice, tea only**;
sugar/oil/flour/dairy/pulses have not been re-gated at 60 days.

| Category | 30-day eligible | 60-day eligible | Change |
|---|---|---|---|
| ghee | 169/561 (30.1%) | 189 | +11.8% |
| rice | 174/671 (25.9%) | 197 | +13.2% |
| tea | 57/206 (27.7%) | 78 | **+36.8%** |

Obs/product within eligible sets (60-day): ghee median 54 (4-62), rice
median 46 (3-62), tea median 56.5 (2-62).

**Tea's rank history:** v1.0.0 (140 products, 57 eligible, 40.7%) treated
tea as uniquely tradeable, likely partly because of a smaller,
less-filtered category (weaker commodity filtering, no Daraz outlier
detection). v2.0.0 at 30 days (206 products, 57 eligible, 27.7%) placed tea
among the mid-ranked categories. At 60 days (242 products, 78 eligible,
32.2%) tea has the largest proportional growth in eligible count of any
category. By flat-rate, tea ranked 1st in v1.0.0, 4th at 30 days and tied
2nd at 60 days.

## Baseline model results (see `01_baseline_forecast_model__v2.0.0_pre-filter-update_jul16-sep15.md` for full detail)

| Category | Beats naive? | Improvement vs rolling7 |
|---|---|---|
| ghee | ✅ | 42.4% |
| rice | ✅ | 31.4% |
| tea | ✅ | 21.9% |
| sugar | ❌ (naive wins) | 2.6% (misleading; see baseline document) |

Sugar's exclusion from `SELECTED_CATEGORIES` in the gated notebook is
consistent with this result.

## Ingestion / cleaning pipeline (context)

v1.0.0 had no outlier detection on Daraz, one flat Naheed
`MAX_UNIT_PRICE`. v2.0.0 (`utils/` refactor) added `products.csv` join,
missing-unit-field rejection, and median-based validation
(`OUTLIER_MULTIPLIER=3.5`, `LOWER_BOUND_MULTIPLIER=0.3`,
`MIN_SAMPLE_FOR_MEDIAN=10`). Row/rejection counts are not directly
comparable across versions as the same measurement: v2.0.0 catches
error classes v1.0.0 structurally could not detect.

## Current category status summary

| Category | Aggregate forecasting | Per-product modeling | Notes |
|---|---|---|---|
| **ghee** | ✅ Use | ✅ Use (189 eligible) | Strongest overall, stable across every check |
| **rice** | ✅ Use | ✅ Use (197 eligible) | Strongest overall, stable across every check |
| **tea** | ✅ Use | ✅ Use (78 eligible) | Solid; rank differed at each checkpoint (see per-product eligibility) |
| sugar | ❌ Excluded (fails vs naive) | Provisional: best 30-day eligibility (35.3%), not re-gated at 60 days | Not re-gated at 60 days |
| oil | ❌ Excluded (sample floor not applied) | Not evaluated | Instability on 4 metrics |
| flour | ❌ Excluded (sample floor not applied) | Not evaluated | Same pattern as oil; first seen at this checkpoint |
| pulses | Not yet modeled | Not evaluated | Nonzero-rate jump is a behavioral shift (stable); weak flat-rate (68.4%) |
| dairy | Not yet modeled | Not evaluated | Weakest flat-rate; `median_obs` 11.0 at both checkpoints, cause not established |

## Open items

1. The `MIN_SAMPLE_FOR_DAILY_MEDIAN` floor has not been applied to oil and
   flour; the four instability metrics have not been re-tested with one.
2. The per-product eligibility gate has been run at 60 days only for
   ghee, rice and tea, not for sugar, oil, flour, dairy or pulses.
3. Rating/review correlation with price movement has not been tested
   (open since the 30-day checkpoint).
4. Dairy's `median_obs` is 11.0 at both checkpoints while every other
   category grew; the cause is not established.
5. The discount-attributed share of spikes (44.1%→32.7%→34.4%) has no
   established trend.
6. `day_of_week`/`is_weekend` have not been tested; at ~8-9 weekly cycles
   at 60 days, history is still short of a confident threshold.