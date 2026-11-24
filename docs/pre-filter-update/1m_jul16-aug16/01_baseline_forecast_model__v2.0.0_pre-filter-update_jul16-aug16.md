# Baseline Forecast Model — Spec

Status: buildable on `pipeline_version=2.0.0` data at the current depth
(~30-63 days). This is a naive, short-lag baseline, not a final model. It
produces an error figure that a later model has to improve on. Per-product
forecasting is out of scope for this phase.

Sources:

- This document has no results table. The month-1 baseline results for this
  specification are in `results/pre-filter-update/cp1_1m_jul16-aug16/baseline/`.
- The std, nonzero-rate and obs/product figures are from the first month of
  pipeline 2.0.0 data (2026-07-16 to 2026-08-16). No notebook output saved in
  this repository reproduces them.
- Figures marked v1.0.0 are from the earlier repository, `pk-ecommerce-price-analysis`.

## Scope

**Grain:** category-aggregate (daily median per category), not per-product.
Per-product modeling is out of scope; the reasons are in
`02_future_data_and_revalidation__v2.0.0_pre-filter-update_jul16-aug16.md`.

**Categories included:** ghee, rice, sugar, tea.
**Categories excluded:** oil (std nearly doubled between the first and second
half of the available window, 0.0982 -> 0.1826, and was already the largest
std of any category in the full window at 0.1455; median obs/product = 8.0,
the lowest of the larger categories); pulses (nonzero-day rate rose from
50% to 100% between halves; the cause is not established); flour, dairy
(weaker and noisier evidence than ghee/rice/sugar/tea; held for the 60-day
recheck).

## Features cleared for this phase

| Feature | Grain | Basis |
|---|---|---|
| `cat_price_lag_1`, `cat_price_lag_2`, `cat_price_lag_7` | daily | Full coverage at current depth (>=7 days per category, no structural NaN gap) |
| `cat_rolling_mean_7` | daily | `min_periods=1` populates it from day 1; by ~day 7 it reflects a full 7-day window |
| `cat_price_change_pct` | daily | Used in the category-aggregate stability check (see the revalidation document) |
| `is_discounted`, `discount_pct`, `has_discount` | any | **Primary feature.** Daraz discount behavior is independent of category selection: `no_discount -> discount` is a price drop in 100% of cases by definition; `discount -> discount` is a price drop in ~89.7% of transitions (v1.0.0) and consistently negative (v2.0.0), so discounts deepen rather than persist. The pattern holds across two pipeline versions and sample sizes and is the strongest-evidenced feature in the dataset. |
| `rolling_discount_rate` (7-window) | daily | Proxy for how promotion-heavy a product has been recently; not yet correlation-tested against price drops directly |

## Features withheld this phase

| Feature | Basis |
|---|---|
| `price_lag_14`, `cat_rolling_mean_14`/`rolling_std_7`-on-few-points | Structurally thin/NaN at current history depth (median obs/product 8-27 depending on category, well under 14) |
| `hour`, `day_of_week`, `is_weekend` | `hour` (session timing) showed no effect in three v1.0.0 checks (category median gap ~0%, paired Wilcoxon p>0.05 in every category, discount-conditioned 46.4% coin-flip); it was not re-tested in v2.0.0 aggregate form at this checkpoint. `day_of_week`/`is_weekend` are untested and need at least two full weekly cycles in the training window |
| `week`, `month` | No variance at current data volume: `month` is near-constant, `week` spans only ~4-9 values depending on window used |
| `price_accel` (2nd difference) | Not computed in the gated pipeline; a noisier version of `price_change` with the same discount confound |
| Per-product lags/rolling stats for any specific product | The per-product gate (nonzero CV + recency filter) has been run once, at this depth; its stability across time windows is not established (see the revalidation document) |
| Oil (any feature) | Category excluded this phase (see Scope) |

## Model structure

Simple, interpretable and evaluable; not the target architecture:

```
target: unit_price (next-day)
features: cat_price_lag_1, cat_price_lag_2, cat_price_lag_7, cat_rolling_mean_7,
          is_discounted, discount_pct, rolling_discount_rate
grain: one row per (category, obs_date); one model per category
```

Implementation: ordinary least squares (`LinearRegression`), one model per
category, in `notebooks/data_preprocessing_and_baseline_model.ipynb`.

One model per category (ghee/rice/sugar/tea) is specified rather than a
pooled model: category volatility differs (std range 0.0135-0.0654 in the
full 31-day window), and pooling would let the noisier categories dominate
the loss.

## Baselines computed alongside the model

1. **Naive persistence:** `forecast(t+1) = price(t)`
2. **7-day rolling mean:** `forecast(t+1) = cat_rolling_mean_7(t)`
3. **Discount-aware naive:** persistence, but if `is_discounted` flips
   on at t+1 relative to t, apply the empirical median drop instead

MAE and RMSE are logged per category for the three baselines and the
feature-based model. Success criterion at this phase: the feature model
beats baseline 2 (rolling mean) by a non-trivial margin consistently across
ghee/rice/sugar/tea, and not only baseline 1.

## Out of scope for this phase

- A final category or per-product selection is not committed on the basis
  of this model's output; all results are provisional (the recheck is
  defined in `02_future_data_and_revalidation__v2.0.0_pre-filter-update_jul16-aug16.md`).
- The model's forecasts are not used as decision-driving output; the model
  is an evaluation harness and benchmark-setter.