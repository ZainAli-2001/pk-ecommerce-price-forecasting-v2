# Category Diagnostics: Months 1–3

Pipeline version `2.0.0`, post commodity-filter update (filter history is in the pipeline repository's git history). Daraz only, daily grain (one row per product per day), all eight categories with no pre-selection. Produced by `notebooks/category_diagnostics.ipynb`. The month-1, month-2 and month-3 outputs were generated on 2026-10-18 from data truncated at each month-end; the month-4 outputs on 2026-11-20. `utils/` is imported from the live pipeline repository, so each run used the filter version current on its run date. Data from 2026-07-16 to 2026-10-15. Companion document: `baseline_model_months_1-3_comparison__v2.0.0_post-filter-update.md`.

## Metrics

- `n_products`: products with at least one daily row. `pct_flat`: share of products whose `unit_price` never varies. `median_obs`: median number of days a product is observed.
- Noise ratio: standard deviation of the day-over-day % change of the all-listings daily median, divided by the same quantity for the median of URLs present on at least 80% of days. A ratio near or below 1 means the category median's movement is not inflated by which listings appear on a given day; well above 1 means it is. The number of URLs in the balanced set is given in parentheses; † marks ratios computed from fewer than 30 URLs.
- Volatility: standard deviation of the day-over-day % change of the daily category median, over the cumulative window.
- Span ratio: maximum divided by minimum `unit_price` in the daily file.
- Day/night gap: (evening median − morning median) / morning median of session-level `unit_price`, sessions at about 06:00 and 18:00 PKT.

## Panel statistics

### Products (`n_products`)

| Category | Month 1 | Month 2 | Month 3 |
|---|---|---|---|
| ghee | 754 | 804 | 836 |
| rice | 731 | 790 | 891 |
| pulses | 539 | 557 | 620 |
| sugar | 111 | 124 | 167 |
| dairy | 204 | 233 | 284 |
| oil | 582 | 657 | 733 |
| flour | 101 | 116 | 125 |
| tea | 705 | 893 | 1,021 |

### Share of products with no price variation (`pct_flat`)

| Category | Month 1 | Month 2 | Month 3 |
|---|---|---|---|
| ghee | 71.5% | 68.2% | 65.8% |
| rice | 65.5% | 60.4% | 59.6% |
| pulses | 79.0% | 70.9% | 67.9% |
| sugar | 71.2% | 67.7% | 72.5% |
| dairy | 77.9% | 75.5% | 72.9% |
| oil | 67.7% | 64.2% | 65.2% |
| flour | 82.2% | 78.5% | 70.4% |
| tea | 89.2% | 88.0% | 88.4% |

### Median days observed per product (`median_obs`)

| Category | Month 1 | Month 2 | Month 3 |
|---|---|---|---|
| ghee | 13.0 | 19.0 | 22.5 |
| rice | 13.0 | 18.0 | 20.0 |
| pulses | 27.0 | 51.0 | 62.5 |
| sugar | 23.0 | 34.5 | 24.0 |
| dairy | 9.5 | 11.0 | 7.0 |
| oil | 9.0 | 14.0 | 12.0 |
| flour | 10.0 | 15.5 | 17.0 |
| tea | 3.0 | 3.0 | 3.0 |

From month 1 to month 3, `pct_flat` fell in 7 of 8 categories and rose in sugar (71.2% → 72.5%). Median days observed per product rose in 6 categories, fell in dairy (9.5 → 7.0), and was unchanged in tea (3.0). Tea's product count rises from 705 to 1,021 while its median days observed stays at 3.0 and `pct_flat` stays between 88.0% and 89.2%.

## Noise ratio (all-listings median vs fixed-URL median)

Balanced set: URLs present on at least 80% of days. Ratio (number of URLs in the balanced set).

| Category | Month 1 | Month 2 | Month 3 |
|---|---|---|---|
| ghee | 1.09 (176) | 1.08 (193) | 1.12 (207) |
| rice | 1.65 (197) | 1.79 (166) | 0.86 (166) |
| pulses | 1.67 (299) | 1.83 (288) | 1.97 (280) |
| sugar | 1.65 (48) | 0.98 (45) | 1.16 (45) |
| dairy | 1.97 (66) | 2.74 (57) | 2.23 (56) |
| oil | 1.97 (95) | 2.37 (71) | 1.45 (67) |
| flour | 0.24 (20)† | 1.84 (18)† | 0.75 (10)† |
| tea | 2.24 (102) | 3.04 (96) | 4.25 (92) |

Range across months: ghee 1.08–1.12; rice 0.86–1.79; pulses 1.67–1.97; sugar 0.98–1.65; dairy 1.97–2.74; oil 1.45–2.37; flour 0.24–1.84 (fewer than 30 URLs in every month); tea 2.24–4.25.

At the stricter 95% threshold the balanced set is often small; for example sugar in month 2 has 20 URLs and a ratio of 143.11.

## Volatility of the daily category median

| Category | Month 1 | Month 2 | Month 3 |
|---|---|---|---|
| ghee | 0.0193 | 0.0170 | 0.0161 |
| rice | 0.0109 | 0.0153 | 0.0156 |
| pulses | 0.0257 | 0.0275 | 0.0256 |
| sugar | 0.1209 | 0.1154 | 0.1143 |
| dairy | 0.0192 | 0.0368 | 0.0345 |
| oil | 0.1162 | 0.1304 | 0.2577 |
| flour | 0.0424 | 0.0967 | 0.0902 |
| tea | 0.0293 | 0.0272 | 0.0240 |

- Oil: 0.1162, 0.1304, 0.2577 in months 1–3 (rising each month).
- Sugar: 0.1209, 0.1154, 0.1143 in months 1–3.
- Flour: 0.0424, 0.0967, 0.0902 in months 1–3.
- Rice, ghee, pulses, tea and dairy: between 0.0109 and 0.0368 in every month.

## Price spread

| Category | Month 1 | Month 2 | Month 3 |
|---|---|---|---|
| ghee | 8.25 | 10.53 | 10.53 |
| rice | 12.22 | 12.35 | 13.06 |
| pulses | 18.50 | 18.50 | 18.50 |
| sugar | 10.38 | 10.38 | 14.61 |
| dairy | 8.86 | 8.86 | 12.83 |
| oil | 17.99 | 24.98 | 34.72 |
| flour | 16.68 | 18.16 | 20.39 |
| tea | 10.10 | 10.90 | 10.90 |

Oil's span ratio is 17.99, 24.98, 34.72 in months 1–3 (rising each month).

## Day/night gap

| Category | Month 1 | Month 2 | Month 3 |
|---|---|---|---|
| ghee | +0.00% | +0.00% | +0.00% |
| rice | +0.11% | +0.71% | +0.00% |
| pulses | -2.80% | -2.82% | -1.36% |
| sugar | -2.50% | -2.94% | -0.78% |
| dairy | +0.00% | +0.00% | +0.00% |
| oil | -0.58% | +0.00% | +0.00% |
| flour | +0.00% | +0.22% | +0.00% |
| tea | +2.37% | +1.29% | +1.19% |

Dairy, flour, ghee and rice stay within ±0.71% in every month; oil within ±0.58%. Pulses (-2.82% to -1.36%), sugar (-2.94% to -0.78%) and tea (+1.19% to +2.37%) have larger gaps. Only median gaps are computed in this document.

## Product-name relevance by category

Source: raw `products_rows.csv` (8,401 products, including those first seen before pipeline 2.0.0; not in the repo), commodity filter not applied. "Name contains a category term" is a regular-expression match on the product name against the category's term list (listed below the table). Product counts include Naheed-sourced rows: oil 132, flour 49, rice 23, ghee 18.

| Category | Daraz scrape keyword(s) | Products | Name contains a category term |
|---|---|---|---|
| tea | `tapal danedar` | 3,030 | 14.1% |
| dairy | `olpers milk` | 648 | 66.8% |
| sugar | `sugar` | 660 | 74.7% |
| oil | `cooking oil` | 1,114 | 97.8% |
| flour | `atta flour` | 398 | 98.7% |
| rice | `basmati rice` | 997 | 99.6% |
| pulses | `daal chana`, `daal masoor` | 656 | 99.8% |
| ghee | `desi ghee` | 898 | 100.0% |

Term lists: tea: tea, chai, danedar, tapal, lipton, patti, chaye, karak, tezdum, family mixture, dahleez, bengal, zordar, harmony, yellow label. Dairy: milk, doodh, cream, butter, dairy, yogurt, dahi, cheese, lassi. Sugar: sugar, cheeni, shakar, shakkar. Oil: oil, tail, canola, sunflower, corn oil, mustard, olive. Flour: atta, flour, maida, suji, besan. Rice: rice, chawal, basmati, sella, biryani. Pulses: daal, dal, chana, masoor, moong, mash, lobia, rajma, lentil, pulses. Ghee: ghee, banaspati, vanaspati, cooking oil, desi ghee.

In a 30-name sample of the 167 sugar names without a term, most are sweeteners outside the term list (jaggery/gur, stevia, monk fruit, molasses); a few are unrelated (for example Epsom salt, a hob set, ispaghol husk).

## Category status after month 3

| Category | Status | Basis |
|---|---|---|
| ghee | Included | Volatility 0.0161; noise ratio 1.12 (207 URLs); beat naive in months 1–2, missed in month 3 |
| rice | Included | Volatility 0.0156; noise ratio 0.86 (166 URLs); beat naive in months 1–2, missed in month 3 |
| pulses | Included | Volatility 0.0256; noise ratio 1.97 (280 URLs); beat naive in all three months |
| sugar | Excluded | Volatility 0.1143, about 4–11 times that of rice, ghee and pulses in each month; passed naive in months 2–3 |
| dairy | Excluded | Missed naive in month 1; volatility 0.0345; noise ratio 2.23 (56 URLs) |
| oil | Excluded | Volatility 0.1162, 0.1304, 0.2577 (rising each month); span ratio 17.99 → 34.72 |
| flour | Excluded | Volatility 0.0902; balanced set under 30 URLs in every month |
| tea | Excluded | 14.1% of product names contain a tea term; median days observed 3.0 in every month |

## Source files

- Panel, noise ratio, volatility, price spread and day/night gap: `panel_stats`, `stability_check`, `category_volatility_full`, `price_spread` and `day_night_median_gap` CSVs in `results/post-filter-update/cp1_1m/diagnostics/`, `results/post-filter-update/cp2_2m/diagnostics/`, `results/post-filter-update/cp3_3m/diagnostics/`.
- Product-name relevance: raw `products_rows.csv` (not in the repo).
- Tea: `results/investigations/tea_contamination/`.