# Baseline Forecast Model: Months 1–4

Pipeline version `2.0.0`, post commodity-filter update. Daraz only. Data from 2026-07-16 to 2026-11-15. Companion document: `category_diagnostics_months_1-4_comparison__v2.0.0_post-filter-update.md`.

## Method

- Series: daily median `unit_price` per category (PKR per kg or per litre), after the commodity filter, with the morning and evening sessions collapsed to one row per product per day.
- The model and the baselines are evaluated at each month-end on the cumulative window (month 1, months 1–2, months 1–3, months 1–4).
- Baselines: naive persistence (previous day's value), 7-day rolling mean (rolling7), and discount-aware naive.
- Pass condition: model MAE below naive-persistence MAE. Margin vs naive is (naive MAE − model MAE) / naive MAE. Improvement vs rolling7 is shown for reference. For every category and month evaluated except sugar, rolling7 has a higher MAE than naive, so improvement vs rolling7 is larger than margin vs naive.
- Categories are selected from the diagnostics output and set in `CLEARED_CATEGORIES` in `notebooks/data_preprocessing_and_baseline_model.ipynb`. Per-product forecasting is not part of this work.

## Results by month

### Month 1

| Category | Model MAE | Naive MAE | Rolling7 MAE | Margin vs naive | Beats naive | Improvement vs rolling7 |
|---|---|---|---|---|---|---|
| ghee | 35.351 | 38.872 | 76.491 | +9.1% | yes | 53.8% |
| rice | 2.670 | 3.808 | 5.320 | +29.9% | yes | 49.8% |
| pulses | 9.265 | 10.928 | 12.226 | +15.2% | yes | 24.2% |
| dairy | 6.774 | 6.035 | 8.430 | -12.2% | no | 19.6% |

Evaluated: rice, ghee, pulses, dairy. Dairy's model MAE is above its naive MAE (6.774 vs 6.035).

### Month 2

| Category | Model MAE | Naive MAE | Rolling7 MAE | Margin vs naive | Beats naive | Improvement vs rolling7 |
|---|---|---|---|---|---|---|
| ghee | 35.046 | 39.473 | 62.953 | +11.2% | yes | 44.3% |
| rice | 4.413 | 4.849 | 6.437 | +9.0% | yes | 31.4% |
| pulses | 11.839 | 12.424 | 13.234 | +4.7% | yes | 10.5% |
| sugar | 38.138 | 40.191 | 39.197 | +5.1% | yes | 2.7% |

Evaluated: rice, ghee, pulses, sugar. Sugar's model MAE is below its naive MAE by 5.1%; its rolling7 MAE (39.197) is below its naive MAE (40.191).

### Month 3

| Category | Model MAE | Naive MAE | Rolling7 MAE | Margin vs naive | Beats naive | Improvement vs rolling7 |
|---|---|---|---|---|---|---|
| ghee | 35.534 | 35.115 | 52.212 | -1.2% | no | 31.9% |
| rice | 4.918 | 4.648 | 6.866 | -5.8% | no | 28.4% |
| pulses | 11.194 | 11.534 | 13.809 | +2.9% | yes | 18.9% |
| sugar | 37.395 | 40.954 | 39.096 | +8.7% | yes | 4.4% |

Evaluated: rice, ghee, pulses, sugar. Rice and ghee have a model MAE above naive for the first time.

### Month 4

| Category | Model MAE | Naive MAE | Rolling7 MAE | Margin vs naive | Beats naive | Improvement vs rolling7 |
|---|---|---|---|---|---|---|
| ghee | 35.688 | 36.183 | 51.009 | +1.4% | yes | 30.0% |
| rice | 7.006 | 6.206 | 8.456 | -12.9% | no | 17.1% |
| pulses | 12.594 | 12.762 | 14.978 | +1.3% | yes | 15.9% |

Evaluated: rice, ghee, pulses. Rice's model MAE is above its naive MAE. Ghee and pulses beat naive by 1.4% and 1.3%.

## Naive baseline over the final month of the month-3 window

Source: `category_daily_baseline_features.csv` of the month-3 checkpoint. First window: 2026-07-16 to 2026-09-14. Final window: 2026-09-15 to 2026-10-15. Naive error is |price − previous day's price| on the category median series.

| Category | Naive MAE, first two months | Naive MAE, final month | Change | Days with a price change, first two months | Days with a price change, final month |
|---|---|---|---|---|---|
| ghee | 40.097 | 25.471 | -36.5% | 96.7% | 87.1% |
| pulses | 12.039 | 10.558 | -12.3% | 85.0% | 87.1% |
| rice | 4.563 | 4.811 | +5.4% | 88.3% | 67.7% |
| sugar | 40.385 | 42.056 | +4.1% | 91.7% | 74.2% |

- Ghee: naive MAE fell 36.5% and the share of days with a price change fell from 96.7% to 87.1%. Naive persistence was a stronger baseline in the final month. Ghee's model MAE on the cumulative window was 35.046 at month 2 and 35.534 at month 3.
- Rice: naive MAE rose 5.4% while the share of days with a price change fell from 88.3% to 67.7%. The change in naive MAE seen for ghee does not appear for rice.
- Pulses: naive MAE fell 12.3%; the model still beat naive at month 3.
- Sugar: naive MAE rose 4.1%.

## Margin vs naive by month

| Category | Month 1 | Month 2 | Month 3 | Month 4 |
|---|---|---|---|---|
| pulses | +15.2% | +4.7% | +2.9% | +1.3% |
| ghee | +9.1% | +11.2% | -1.2% | +1.4% |
| rice | +29.9% | +9.0% | -5.8% | -12.9% |
| sugar | not evaluated | +5.1% | +8.7% | not evaluated |
| dairy | -12.2% | not evaluated | not evaluated | not evaluated |

Margin vs naive (positive = model better). "not evaluated" means the category was not in `CLEARED_CATEGORIES` for that run.

Observations:

- Rice: margin vs naive falls each month and is negative in months 3 and 4. Naive MAE by month: 3.808 → 4.849 → 4.648 → 6.206. Model MAE by month: 2.670 → 4.413 → 4.918 → 7.006.
- Ghee: positive in months 1, 2 and 4; negative in month 3.
- Pulses: positive in all four months; the margin falls each month from 15.2% to 1.3%.
- Sugar: evaluated in months 2 and 3 only; positive in both.
- Dairy: evaluated in month 1 only; negative.

## Comparability across months

The daily category series for rice, ghee and pulses (`category_daily_baseline_features.csv`) are identical on every date shared by the month-2, month-3 and month-4 exports. The month-1 and month-2 exports differ on one date, 2026-08-16, the final date of the month-1 export. Month-4 results for these three categories are therefore comparable with months 2 and 3.

## Category decisions after month 4

| Decision | Categories | Basis |
|---|---|---|
| Included | ghee, pulses | Ghee beat naive in 3 of 4 months (miss in month 3, when naive MAE fell 36.5% over the final month). Pulses beat naive in all 4 months; month-4 margins are 1.4% (ghee) and 1.3% (pulses). |
| Provisional | rice | Beat naive in months 1 and 2, missed in months 3 and 4. Provisional means kept in `CLEARED_CATEGORIES` and not classed as included. |
| Excluded | sugar | Passed naive in the two months evaluated (5.1%, 8.7%). Excluded on diagnostics evidence: daily-median std of 0.1143–0.2009, month-4 coverage and balanced-panel size (see companion document). |
| Excluded | dairy | Missed naive in month 1, the only month evaluated. |
| Excluded | oil, flour, tea | Not evaluated. Evidence in the companion document. |

## Source files

- Results tables: `baseline_model_results.csv` in `results/post-filter-update/cp1_1m/baseline/`, `results/post-filter-update/cp2_2m/baseline/`, `results/post-filter-update/cp3_3m/baseline/`, `results/post-filter-update/cp4_4m/baseline/`.
- Final-month analysis: `category_daily_baseline_features.csv` in `results/post-filter-update/cp3_3m/baseline/`.
- Comparability: `category_daily_baseline_features.csv` across `cp1_1m`–`cp4_4m`.