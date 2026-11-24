# Feature Extraction Reference — What, How, Why, Status

This document lists every feature considered across the v1.0.0 forecasting
reference, the v2.0.0 numeric comparison and the pipeline refactor: what it
is, how it is computed, the evidence for or against using it, and its
current status. The checks that change a status at the next milestone are
defined in `02_future_data_and_revalidation__v2.0.0_pre-filter-update_jul16-aug16.md`.

Sources:

- Figures are from the first month of pipeline 2.0.0 data (2026-07-16 to
  2026-08-16) and, where marked v1.0.0, from the earlier repository
  `pk-ecommerce-price-analysis`. No notebook output saved in this
  repository reproduces the month-1 figures.
- The Naheed session coverage for pipeline 2.0.0 (63 of 315 sessions,
  20.0%) is printed in `notebooks/pre-filter-update/forecasting_stage2.ipynb`,
  cell 9. The saved run uses the later commodity filter and gives the same
  value.

## Data sources and grain

- `product_prices` — append-only time series, one row per scrape
  observation, `pipeline_version` column (`1.0.0` backfilled, `2.0.0`
  for new rows going forward).
- `products` — deduped by `url`, product identity persists across
  pipeline versions.
- Session grain: ~2 scrapes/day (6AM/6PM PKT, cron). Daily grain: two
  sessions collapsed into one row per product-day (used because no
  day/night effect was found; see Calendar features).
- Daraz only. Naheed is captured but excluded from forecasting (see
  Cross-source features).

## Target

| Feature | Notes |
|---|---|
| `unit_price` | PKR per kg or per liter. Models are built per `unit_type`; kg and liter series are not mixed in one model. |

## Identifiers (not features)

`url`, `name`, `category`, `source`, `unit_type`, `unit_value`, `run_id`,
`scraped_at`/`obs_date`, `scraper_version`, `pipeline_version`. Used for
grouping/joining/filtering only. `category`/`unit_type` may be used as
categorical splits.

## Calendar features

| Feature | Status | Evidence |
|---|---|---|
| `hour` / day-vs-night split | **Not used: no effect found** | Three independent checks: (1) category median gap morning-vs-evening ranged -3.23% (sugar) to +1.79% (tea), mostly ~0%; (2) paired same-product Wilcoxon p>0.05 in every category (flour 0.77, ghee 0.12, oil 0.93, pulses 0.63, rice 0.57, tea 0.80); (3) discount-conditioned check — restricted to `no_discount->no_discount` transitions, evening-vs-morning direction was a 46.4% coin flip, mean change ~0 relative to price levels. Apparent day/night patterns are discount timing, not a base-price effect. This is the reason for the daily collapse. |
| `day_of_week`, `is_weekend` | Untested | Needs >=2 full weekly cycles in the training window; not enough calendar span yet (revalidation document, check 6). |
| `week`, `month` | Not usable: no variance | `month` near-constant, `week` spans too few distinct values at current volume. Needs multiple months of history. |

## Lag features

| Feature | Grain | Status |
|---|---|---|
| `price_lag_1` | daily only | Usable. At session grain, 95-100% of consecutive pairs are byte-identical (no signal), which is why daily grain is used for lag features. |
| `price_lag_2`, `price_lag_7` | daily | Usable |
| `price_lag_14` | daily | Provisional: NaN for the majority of products at current depth (median obs/product ~7-27 depending on category, under 14). Computed and excluded from modeling at this checkpoint; rechecked at the 60-day milestone. |

## Rolling statistics

| Feature | Status |
|---|---|
| `rolling_mean_7`/`min_7`/`max_7` | Usable once a product has close to a full 7-obs window; `min_periods=1` populates it from obs #1, but early values are not a reliable "typical price." |
| `rolling_mean_14` | Same provisional status as `price_lag_14` |
| `rolling_std_7` | Same caveat as rolling_mean_7, stronger: a std from 1-2 points is close to meaningless; not relied on below ~4-5 observations |

## Momentum features

| Feature | Status |
|---|---|
| `price_change`, `price_change_pct` | Usable at daily grain. Most non-zero changes are discount events rather than organic movement, so they are read together with `has_discount`/`discount_pct`. |
| `price_accel` | Not computed in the gated pipeline. Noisier, with the same discount confound doubled. |

## Discount features — strongest validated signal in the dataset

| Feature | Status | Evidence |
|---|---|---|
| `is_discounted`, `discount_pct`, `discount_rate` | **Primary candidate, high confidence** | `no_discount -> discount`: 100% price drop by definition. `discount -> discount`: ~89.7% (v1.0.0), consistently negative in v2.0.0, so active discounts deepen rather than persist. Structural Daraz behavior, computed across the full dataset regardless of category selection. Confirmed across two pipeline versions and sample sizes (v1.0.0: ~14k daily rows; v2.0.0: ~48k daily rows). |
| `has_discount`, `rolling_discount_rate` | Usable | 7-window discount frequency, a proxy for how promotion-heavy a product has been recently. Not yet correlation-tested against subsequent price drops. |

## Rating / review features

| Feature | Status |
|---|---|
| `has_rating`, `has_review` | Usable as auxiliary/popularity-proxy features. Coverage ~49-62% depending on pipeline version/source; the missingness is stable enough to use as-is. |
| `rating_clean`, `review_count_clean`, `log_review_count`, `rating_weighted` | Computed and available, untested for price correlation; exploratory only |

## Spike detection

| Feature | Status |
|---|---|
| `is_spike` | Used as a **label**, not an input feature. `rolling(14, min_periods=5)` needs 5+ prior obs to fire, so it is `0` by construction for short-history products and is not a genuine "no spike" signal for them. Rare event (~1.16-1.17% of daily rows, stable across pipeline versions and a 3.4x volume increase). Discount-driven share fell from 44.1% (v1.0.0) to ~32.7% (v2.0.0), so more non-discount-driven spikes appear with more data; the cause is not established. |

## Cross-source (Naheed) features

| Feature | Status |
|---|---|
| `price_gap`/`price_gap_pct` (Daraz vs Naheed) | **Dropped.** Even with correct session-clustering, only ~20% of Naheed sessions found a Daraz match within a 3-hour tolerance (46/230 in v1.0.0, 63/315 in v2.0.0). The rate is consistent across versions, indicating a structural scrape-timing mismatch rather than a measurement error. Rule of thumb: cross-store features need >=30-50% temporal coverage to be trustworthy; 20% is below the floor. Naheed also disqualifies itself on other grounds: 71.1% flat-price rate (vs Daraz's ~56-68% depending on grain), 0% rating/review coverage. |

## Category-aggregate features (v2.0.0 addition)

Built by grouping daily grain by `(category, obs_date)`, taking the
median, then computing lag/rolling/change features on that series —
same feature shapes as per-product (`cat_price_lag_1/7`,
`cat_rolling_mean_7`, `cat_price_change_pct`) but at category grain.
This feeds category-as-a-whole forecasting for categories that
don't have enough per-product eligibility.

Category-aggregate stability and per-product flat-price rate measure
different things. A category can have a low per-product flat-rate (most
products individually move) but a stable aggregate, or the reverse. Dairy
is the clearest example: 74.5% flat-price at the product level (one of the
worst) but real day-to-day movement in `cat_price_change_pct` (daily
category swings like +1.9%, +2.5%, -2.0%, +3.3% observed in a sample). The
aggregate's own std and nonzero-day rate are therefore checked directly
rather than inferred from the per-product table.

30-day std / nonzero-day-rate by category (full window):

| Category | std | pct_nonzero | Status |
|---|---|---|---|
| oil | 0.1455 | 100.0% | **Disqualified.** Attributed to thin-sample compositional noise (min obs/product=8.0); single-day swings of -33.7% to +55.9% in `.describe()`; std rose between window halves (0.0982 -> 0.1826) |
| sugar | 0.0654 | 87.5% | Usable; stable across halves (0.0716 -> 0.0612) |
| flour | 0.0422 | 84.4% | Usable, weaker evidence; nonzero% fell 100%->68.75% between halves |
| dairy | 0.0386 | 90.6% | Usable; moderate drift between halves |
| tea | 0.0304 | 96.9% | Usable; most consistent nonzero rate, only category with nonzero median (not just mean) |
| pulses | 0.0257 | 75.0% | **Hold.** nonzero% rose 50%->100% between halves; cause not established |
| ghee | 0.0196 | 93.8% | Usable; most stable std across halves (0.0192 -> 0.0200) |
| rice | 0.0135 | 90.6% | Usable with caution; std grew 3.3x between halves (0.0055 -> 0.0181) |

## Per-product eligibility gate

Two-step filter applied independently per category, used to decide
which individual products get their own time-series model vs falling
back to the category aggregate:

1. **Nonzero variation:** keep products with `CV > 0` (not flat-price
   across their whole history)
2. **Recency:** of those, drop any product with no observation in the
   last 3 days of the window

30-day results (daily grain, v2.0.0):

| Category | Total products | Eligible | Eligible % |
|---|---|---|---|
| sugar | 133 | 47 | 35.3% |
| ghee | 561 | 169 | 30.1% |
| tea | 206 | 57 | 27.7% |
| rice | 671 | 174 | 25.9% |

Not run for oil, flour, dairy, pulses (excluded from this phase; reasons
in the category-aggregate table above).

**Tea's earlier result:** v1.0.0 ran this gate only for tea (140 products,
57 eligible, 40.7%) and treated tea as uniquely tradeable. At v2.0.0's
larger, better-filtered product set, tea's eligible *rate* fell to
27.7% while the eligible *count* stayed identical (57): the same core 57
tea products keep clearing the gate. Tea's earlier advantage was likely
partly an artifact of a smaller, less-filtered v1.0.0 category (weaker
commodity filtering let premix/tea-bag products in; no outlier detection
existed on Daraz in v1.0.0). Tea is evaluated on the same footing as the
other categories.

## Ingestion / cleaning pipeline (context for why row counts differ across versions)

v1.0.0 rejected on `commodity_filter` and `outlier_unit_price` only, no
master product list, no outlier detection on Daraz specifically
(Naheed had one flat `MAX_UNIT_PRICE=10000` across all categories).
v2.0.0 (`utils/` refactor, see `REFACTOR_SUMMARY.md`) replaces this
with: `products.csv` join (orphan rejection), missing-unit-field
rejection, per-category `MAX_UNIT_PRICE` as a last-resort fallback,
and a **median-based validation** (`OUTLIER_MULTIPLIER=3.5`,
`LOWER_BOUND_MULTIPLIER=0.3`, gated on `MIN_SAMPLE_FOR_MEDIAN=10`) that
checks both high and low ends per batch. v1.0.0 and v2.0.0
row/rejection counts are therefore not directly comparable as the same
measurement: v2.0.0 catches classes of bad data (unit mismatches,
implausibly-cheap mis-parses, non-commodity variants like tea premix)
that v1.0.0 structurally could not detect.

## Open items

1. All tables in this document are at 30 days. The 60-day recomputation and
   its pass/fail criteria are defined in
   `02_future_data_and_revalidation__v2.0.0_pre-filter-update_jul16-aug16.md`.
2. Pulses' nonzero% jump (50% to 100% between halves) is unexplained;
   pulses is not included in any model.
3. No `MIN_SAMPLE_FOR_DAILY_MEDIAN` floor is applied in the category-daily
   aggregation step (the analogue of `validation.py`'s
   `MIN_SAMPLE_FOR_MEDIAN`); oil has not been re-tested with one.
4. The per-product eligibility gate has not been run for flour, dairy or
   oil.
5. `is_spike`'s non-discount-driven share grew (discount-attributed share
   44.1% -> 32.7%); the cause is not established.