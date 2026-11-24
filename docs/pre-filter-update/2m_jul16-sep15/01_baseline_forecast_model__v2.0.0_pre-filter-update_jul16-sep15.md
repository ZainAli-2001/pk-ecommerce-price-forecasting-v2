# Baseline Forecast Model — Spec

Status: **evaluated at 60 days (61-day window, 2026-07-16 to 2026-09-15),
`pipeline_version=2.0.0`.** This is a naive, short-lag baseline, not a
final model, with evaluation results (below). `SELECTED_CATEGORIES` in the
gated notebook (`notebooks/pre-filter-update/feature_engineering_gated.ipynb`,
cell 5) is **ghee, rice, tea**. Sugar was evaluated and is not in the final
selection (see Results).

Sources:

- Results table: `results/pre-filter-update/cp2_2m_jul16-sep15/baseline/baseline_model_results.csv`.
- `SELECTED_CATEGORIES`: `notebooks/pre-filter-update/feature_engineering_gated.ipynb`, cell 5.
- Per-product `price_lag_14` and `rolling_mean_14` coverage (67.8-71.4%, 100%):
  computed from `selected_categories_per_product_features.csv`, written by the
  gated notebook and not included in this repository.

## Scope

**Grain:** category-aggregate (daily median per category) for the
model results below. Per-product modeling is gated separately (see
`03_feature_extraction_reference__v2.0.0_pre-filter-update_jul16-sep15.md`,
per-product eligibility section) and has eligible-product sets for
ghee/rice/tea.

**Categories evaluated:** ghee, rice, sugar, tea.
**Categories excluded from evaluation:** oil, flour, pulses, dairy; the
reasons for each are in
`03_feature_extraction_reference__v2.0.0_pre-filter-update_jul16-sep15.md`.
Oil and flour have aggregate series that are not trusted without a
`MIN_SAMPLE_FOR_DAILY_MEDIAN` floor.

## Features used

| Feature | Grain | Status |
|---|---|---|
| `cat_price_lag_1`, `cat_price_lag_7` | daily | Full coverage, used |
| `cat_rolling_mean_7` | daily | Full coverage (`min_periods=1`), used |
| `cat_price_change_pct` | daily | Used, also the basis of the evaluation target |
| `is_discounted`, `discount_pct`, `has_discount`, `rolling_discount_rate` | any | Used. Still the strongest-evidenced feature, structural across both pipeline versions |
| `price_lag_2` | daily | Listed in the specification; the category-aggregate column `cat_price_lag_2` is present in `category_daily_baseline_features.csv` |

Not part of this category-aggregate evaluation: `price_lag_14` and
`rolling_mean_14`-derived features. At the **per-product** grain,
`price_lag_14` coverage is 67.8-71.4% non-null for ghee/rice/tea and
`rolling_mean_14` is 100% non-null (feature reference).
Calendar features (`day_of_week`, `week`, `month`, `hour`) remain
withheld per the day/night and seasonality findings at 60
days (feature reference).

## Results (61-day evaluation window, n=61 per category)

| Category | Model MAE | Naive MAE | Rolling7 MAE | Discount-aware MAE | Beats naive? | Beats rolling7? | Improvement vs rolling7 |
|---|---|---|---|---|---|---|---|
| **ghee** | 35.696 | 40.017 | 62.022 | 42.474 | ✅ | ✅ | **42.4%** |
| **rice** | 4.413 | 4.849 | 6.437 | 4.849 | ✅ | ✅ | **31.4%** |
| tea | 38.495 | 42.167 | 49.311 | 42.255 | ✅ | ✅ | 21.9% |
| **sugar** | 27.814 | **25.281** | 28.565 | 25.412 | ❌ | ✅ (marginal) | 2.6% |

**Pass condition at this checkpoint** (the month-1 specification set the
rolling-mean baseline as the bar): beat **naive persistence**, not only
rolling7. Rolling7 was a weak baseline for some categories (sugar), so
beating rolling7 alone is not sufficient evidence of a working model.

## Verdict per category

- **Ghee, rice: pass.** Large, consistent improvement over all three
  baselines, in line with their stability evidence (lowest, most stable
  category-aggregate std across all window splits tested; see feature
  reference).
- **Tea: pass, moderate.** Beats naive and rolling7 by a real margin. The
  discount-aware baseline (42.255) is close to the full model (38.495), so
  most of tea's forecastable signal appears to come from discount
  features rather than lag/rolling features.
- **Sugar: fail.** Naive persistence (25.281) beats the model (27.814).
  Sugar's aggregate series is not used. Sugar is not in the notebook's final
  `SELECTED_CATEGORIES`. Sugar's per-product signal is assessed separately
  (per-product eligibility section of the feature reference); the failure
  is specific to the aggregate/category-level series.

## Baselines computed alongside the model

1. Naive persistence — `forecast(t+1) = price(t)`
2. 7-day rolling mean — `forecast(t+1) = cat_rolling_mean_7(t)`
3. Discount-aware naive — persistence, adjusted for discount-state
   transitions

All three are reported in the results above for every evaluated category.

## Out of scope

- The model's forecasts are not used as decision-driving output; the model
  is a benchmark-setter, not a production model.
- The aggregate grain is not extended to oil, flour, dairy or pulses until
  their open items in the feature reference are resolved.
- Sugar's aggregate series is not added to any downstream model while the
  naive-persistence failure is unexplained. A possible cause is sugar's
  category-composition volatility: sugar showed the largest and only
  divergent median-gap change between the 30-day and 60-day checks (day/night
  section of the feature reference).