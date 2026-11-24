# Category Diagnostics: Months 1–4

Pipeline version `2.0.0`, post commodity-filter update (filter history is in the pipeline repository's git history). Daraz only, daily grain (one row per product per day), all eight categories with no pre-selection. Produced by `notebooks/category_diagnostics.ipynb`. The month-1, month-2 and month-3 outputs were generated on 2026-10-18 from data truncated at each month-end; the month-4 outputs on 2026-11-20. `utils/` is imported from the live pipeline repository, so each run used the filter version current on its run date. Data from 2026-07-16 to 2026-11-15. Companion document: `baseline_model_months_1-4_comparison__v2.0.0_post-filter-update.md`.

## Metrics

- `n_products`: products with at least one daily row. `pct_flat`: share of products whose `unit_price` never varies. `median_obs`: median number of days a product is observed.
- Noise ratio: standard deviation of the day-over-day % change of the all-listings daily median, divided by the same quantity for the median of URLs present on at least 80% of days. A ratio near or below 1 means the category median's movement is not inflated by which listings appear on a given day; well above 1 means it is. The number of URLs in the balanced set is given in parentheses; † marks ratios computed from fewer than 30 URLs.
- Volatility: standard deviation of the day-over-day % change of the daily category median, over the cumulative window.
- Span ratio: maximum divided by minimum `unit_price` in the daily file.
- Day/night gap: (evening median − morning median) / morning median of session-level `unit_price`, sessions at about 06:00 and 18:00 PKT.

## Panel statistics

### Products (`n_products`)

| Category | Month 1 | Month 2 | Month 3 | Month 4 |
|---|---|---|---|---|
| ghee | 754 | 804 | 836 | 1,023 |
| rice | 731 | 790 | 891 | 1,209 |
| pulses | 539 | 557 | 620 | 639 |
| sugar | 111 | 124 | 167 | 164 |
| dairy | 204 | 233 | 284 | 248 |
| oil | 582 | 657 | 733 | 484 |
| flour | 101 | 116 | 125 | 167 |
| tea | 705 | 893 | 1,021 | 1,096 |

### Share of products with no price variation (`pct_flat`)

| Category | Month 1 | Month 2 | Month 3 | Month 4 |
|---|---|---|---|---|
| ghee | 71.5% | 68.2% | 65.8% | 66.9% |
| rice | 65.5% | 60.4% | 59.6% | 64.8% |
| pulses | 79.0% | 70.9% | 67.9% | 59.8% |
| sugar | 71.2% | 67.7% | 72.5% | 65.2% |
| dairy | 77.9% | 75.5% | 72.9% | 61.7% |
| oil | 67.7% | 64.2% | 65.2% | 59.7% |
| flour | 82.2% | 78.5% | 70.4% | 74.2% |
| tea | 89.2% | 88.0% | 88.4% | 88.4% |

### Median days observed per product (`median_obs`)

| Category | Month 1 | Month 2 | Month 3 | Month 4 |
|---|---|---|---|---|
| ghee | 13.0 | 19.0 | 22.5 | 20.0 |
| rice | 13.0 | 18.0 | 20.0 | 17.0 |
| pulses | 27.0 | 51.0 | 62.5 | 69.0 |
| sugar | 23.0 | 34.5 | 24.0 | 28.0 |
| dairy | 9.5 | 11.0 | 7.0 | 24.0 |
| oil | 9.0 | 14.0 | 12.0 | 16.0 |
| flour | 10.0 | 15.5 | 17.0 | 10.0 |
| tea | 3.0 | 3.0 | 3.0 | 3.0 |

From month 1 to month 4, `pct_flat` fell in 8 of 8 categories. Median days observed per product rose in 6 categories, fell in none, and was unchanged in flour (10.0) and tea (3.0). Tea's product count rises from 705 to 1,096 while its median days observed stays at 3.0 and `pct_flat` stays between 88.0% and 89.2%.

Between months 3 and 4, `n_products` fell for oil (733 → 484), dairy (284 → 248) and sugar (167 → 164). Dairy's median days observed rose from 7.0 to 24.0 and its `pct_flat` fell from 72.9% to 61.7%. See "Month-4 export compared with month-3 export".

## Noise ratio (all-listings median vs fixed-URL median)

Balanced set: URLs present on at least 80% of days. Ratio (number of URLs in the balanced set).

| Category | Month 1 | Month 2 | Month 3 | Month 4 |
|---|---|---|---|---|
| ghee | 1.09 (176) | 1.08 (193) | 1.12 (207) | 1.38 (163) |
| rice | 1.65 (197) | 1.79 (166) | 0.86 (166) | 1.56 (103) |
| pulses | 1.67 (299) | 1.83 (288) | 1.97 (280) | 1.27 (256) |
| sugar | 1.65 (48) | 0.98 (45) | 1.16 (45) | 1.93 (2)† |
| dairy | 1.97 (66) | 2.74 (57) | 2.23 (56) | 6.22 (47) |
| oil | 1.97 (95) | 2.37 (71) | 1.45 (67) | 3.83 (14)† |
| flour | 0.24 (20)† | 1.84 (18)† | 0.75 (10)† | 1.59 (3)† |
| tea | 2.24 (102) | 3.04 (96) | 4.25 (92) | 3.76 (92) |

Range across months: ghee 1.08–1.38; rice 0.86–1.79; pulses 1.27–1.97; sugar 0.98–1.93; dairy 1.97–6.22; oil 1.45–3.83; flour 0.24–1.84 (fewer than 30 URLs in every month); tea 2.24–4.25.

At the stricter 95% threshold the balanced set is often small; for example sugar in month 2 has 20 URLs and a ratio of 143.11.

### Month 4 at three presence thresholds

| Category | ≥60% of days | ≥80% of days | ≥95% of days |
|---|---|---|---|
| ghee | 1.10 (255) | 1.38 (163) | 1.75 (54) |
| rice | 0.89 (205) | 1.56 (103) | 1.02 (29)† |
| pulses | 1.24 (310) | 1.27 (256) | 1.52 (172) |
| sugar | 0.79 (46) | 1.93 (2)† | n/a (0)† |
| dairy | 2.42 (60) | 6.22 (47) | 1.51 (25)† |
| oil | 0.69 (63) | 3.83 (14)† | 7.63 (5)† |
| flour | 1.10 (14)† | 1.59 (3)† | n/a (0)† |
| tea | 2.69 (103) | 3.76 (92) | 4.88 (84) |

For oil the three thresholds give different readings (0.69, 3.83, 7.63) on balanced sets of 63, 14 and 5 URLs. For dairy they give 2.42, 6.22 and 1.51 on 60, 47 and 25 URLs.

## Volatility of the daily category median

| Category | Month 1 | Month 2 | Month 3 | Month 4 |
|---|---|---|---|---|
| ghee | 0.0193 | 0.0170 | 0.0161 | 0.0158 |
| rice | 0.0109 | 0.0153 | 0.0156 | 0.0210 |
| pulses | 0.0257 | 0.0275 | 0.0256 | 0.0287 |
| sugar | 0.1209 | 0.1154 | 0.1143 | 0.2009 |
| dairy | 0.0192 | 0.0368 | 0.0345 | 0.0338 |
| oil | 0.1162 | 0.1304 | 0.2577 | 0.1362 |
| flour | 0.0424 | 0.0967 | 0.0902 | 0.1552 |
| tea | 0.0293 | 0.0272 | 0.0240 | 0.0216 |

- Oil: 0.1162, 0.1304, 0.2577 in months 1–3 (rising each month), 0.1362 in month 4.
- Sugar: 0.1209, 0.1154, 0.1143 in months 1–3, 0.2009 in month 4.
- Flour: 0.0424, 0.0967, 0.0902 in months 1–3, 0.1552 in month 4.
- Rice, ghee, pulses, tea and dairy: between 0.0109 and 0.0368 in every month.

## Price spread

| Category | Month 1 | Month 2 | Month 3 | Month 4 |
|---|---|---|---|---|
| ghee | 8.25 | 10.53 | 10.53 | 10.53 |
| rice | 12.22 | 12.35 | 13.06 | 13.09 |
| pulses | 18.50 | 18.50 | 18.50 | 18.50 |
| sugar | 10.38 | 10.38 | 14.61 | 156.25 |
| dairy | 8.86 | 8.86 | 12.83 | 12.83 |
| oil | 17.99 | 24.98 | 34.72 | 22.21 |
| flour | 16.68 | 18.16 | 20.39 | 58.82 |
| tea | 10.10 | 10.90 | 10.90 | 10.90 |

Oil's span ratio is 17.99, 24.98, 34.72 in months 1–3 (rising each month) and 22.21 in month 4.

Sugar's span ratio is 14.61 in month 3 and 156.25 in month 4; flour's is 20.39 and 58.82. The month-4 minimum `unit_price` is 18.0 for sugar and 34.0 for flour (month 3: 192.5 and 98.0).

## Day/night gap

| Category | Month 1 | Month 2 | Month 3 | Month 4 |
|---|---|---|---|---|
| ghee | +0.00% | +0.00% | +0.00% | -0.13% |
| rice | +0.11% | +0.71% | +0.00% | +0.00% |
| pulses | -2.80% | -2.82% | -1.36% | -1.23% |
| sugar | -2.50% | -2.94% | -0.78% | -2.94% |
| dairy | +0.00% | +0.00% | +0.00% | +0.00% |
| oil | -0.58% | +0.00% | +0.00% | +0.77% |
| flour | +0.00% | +0.22% | +0.00% | +0.00% |
| tea | +2.37% | +1.29% | +1.19% | +1.13% |

Dairy, flour, ghee and rice stay within ±0.71% in every month; oil within ±0.77%. Pulses (-2.82% to -1.23%), sugar (-2.94% to -0.78%) and tea (+1.13% to +2.37%) have larger gaps. Only median gaps are computed in this document.

## Month-4 export compared with month-3 export

A cumulative window built from append-only data under an unchanged filter cannot lose products or rows. Oil did in the month-4 export:

Product-days with a `unit_price`, cumulative:

| Category | Month 1 | Month 2 | Month 3 | Month 4 |
|---|---|---|---|---|
| ghee | 9,072 | 17,226 | 24,850 | 32,228 |
| rice | 9,996 | 18,475 | 26,891 | 34,554 |
| pulses | 11,149 | 21,245 | 30,976 | 40,673 |
| sugar | 1,906 | 3,624 | 5,372 | 5,739 |
| dairy | 2,671 | 4,801 | 6,934 | 9,352 |
| oil | 6,179 | 10,801 | 14,717 | 11,209 |
| flour | 1,020 | 1,843 | 2,493 | 2,842 |
| tea | 3,467 | 6,749 | 10,003 | 13,498 |

Product-days added in each month:

| Category | Month 1 | Month 2 | Month 3 | Month 4 |
|---|---|---|---|---|
| ghee | +9,072 | +8,154 | +7,624 | +7,378 |
| rice | +9,996 | +8,479 | +8,416 | +7,663 |
| pulses | +11,149 | +10,096 | +9,731 | +9,697 |
| sugar | +1,906 | +1,718 | +1,748 | +367 |
| dairy | +2,671 | +2,130 | +2,133 | +2,418 |
| oil | +6,179 | +4,622 | +3,916 | -3,508 |
| flour | +1,020 | +823 | +650 | +349 |
| tea | +3,467 | +3,282 | +3,254 | +3,495 |

Oil's price distribution, month 3 and month 4:

| Month | n | min | p10 | median | p90 | max | span ratio |
|---|---|---|---|---|---|---|---|
| 3 | 14,717 | 360 | 610 | 1,000 | 4,270 | 12,500 | 34.72 |
| 4 | 11,209 | 360 | 605 | 750 | 1,440 | 7,995 | 22.21 |

- Oil: `n_products` 733 → 484, product-days 14,717 → 11,209, volatility 0.2577 → 0.1362. Changes in oil's statistics between months 3 and 4 combine the added month with a change in the underlying data.
- Dairy and sugar lost products (284 → 248 and 167 → 164) while gaining product-days.
- Sugar added 367 product-days in month 4 against 1,906, 1,718 and 1,748 in months 1–3. Flour added 349 against 1,020, 823 and 650. Their balanced sets at month 4 hold 2 and 3 URLs (80% threshold). Sugar's and flour's volatility rises in month 4 (0.1143 → 0.2009; 0.0902 → 0.1552) coincide with the month-4 minimum prices reported under Price spread; the cause has not been isolated.
- The daily category series for rice, ghee and pulses are identical on every date shared by the month-2, month-3 and month-4 exports, so the month-4 baseline results for those three are unaffected.
- The month-1 and month-2 exports differ for these categories on one date, 2026-08-16, the final date of the month-1 export.

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

## Category investigations

### Tea

The tea rows in the tables above are computed on the kept list, in which 71.3% of the 1,148 product names in the latest `tea_kept.csv` contain no tea term. In the raw products table, 14.1% of 3,030 tea product names contain a tea term. Detail is in `results/investigations/tea_contamination/tea_contamination.md`.

### Dairy

Source: raw products and prices tables (not in the repo), months 1–3, commodity filter not applied; name groups are regular-expression heuristics.

- All 648 dairy products were found with the keyword `olpers milk`. 105 names (16.2%) contain "olper".
- 465 names (71.8%) contain a dairy term or "Olper's"; 183 (28.2%) contain neither.

| Name group | Products | Priced products | Median days observed | % flat | Median unit price |
|---|---|---|---|---|---|
| Dairy term or "Olper's" | 465 | 203 | 16 | 64.0% | 592 |
| Other / unclassified | 97 | 57 | 3 | 82.5% | 998 |
| "Ol-" prefix names (Olive, Olivia, Olim, Oliya, Canolive) | 36 | 22 | 6 | 95.5% | 1,454 |
| Milk-based nutrition (Ovaltine, Milo, Cerelac) | 35 | 8 | 6.5 | 62.5% | 2,167 |
| Pet food | 15 | 13 | 2 | 76.9% | 1,362 |

- Of 303 priced dairy products, 203 are in the first group. The other 100 are observed on a median of 2–6.5 days (against 16) and supply 5.8% of price rows and 7.0% of product-days.
- Median number of priced dairy products per day: 80. Standard deviation of the day-over-day change in the daily median: 0.0374 with all priced rows, 0.0294 with the first group only; the largest single-day move is 16.2% in both.
- Share of newly created dairy products that are in the non-dairy groups, by month first seen: June 9.2% (249 products), July 36.5% (222), August 50.5% (91), September 38.7% (31), October 38.2% (55). Pipeline 1.0.0 (June and 1–15 July) fetched 2 pages per keyword; pipeline 2.0.0 (from 2026-07-16) fetches up to 8 pages with a yield-based stop (`MAX_PAGES_PER_PRODUCT`, pipeline repository refactor notes).
- Manual search for `olpers milk` on the Daraz site: pages 1–2 show relevant products; pages 3–5 show fewer genuine milk products mixed with unrelated ones; no relevant milk products beyond page 5.

### Oil

Source: raw products and prices tables (not in the repo), months 1–3, commodity filter not applied, so figures differ from the filtered diagnostics tables above.

- Daily priced oil products: 107–244 (median 168). A minimum of 10 priced products per day (the pipeline's `MIN_SAMPLE_FOR_MEDIAN`) is met on every day.
- Priced oil products per scrape run: median 145, range 105–192; none of 184 runs is below half the median.
- Across 91 consecutive-day pairs, a median of 12% of the previous day's priced oil products are absent from the next day's scrape (maximum 66%, 2026-10-12 → 2026-10-13). In every case the product is absent from the scrape; none appears without a price.
- Across the 75 day-pairs with at least 5 lost and 5 retained products, the lost products were cheaper than the retained ones on 33% of pairs (median lost/retained price ratio 1.21).
- 2026-10-12 → 2026-10-13: 193 priced products, 127 absent the next day (66%). Median price of the 127: 890; of the 66 retained: 3,814. Daily median: 1,453 → 5,612.
- 93.4% of priced oil rows are per litre and 6.6% per kg. The standard deviation of the day-over-day change in the daily median is 0.3512 with all priced rows and 0.3557 with litre rows only.

### Sugar and flour

- Daraz keywords: `sugar` (sugar), `atta flour` (flour). Scraper run statistics (not in the repo) show the commodity filter removing 75–90% of their results, leaving 6–9 priced items per run.
- Raw pull, filter not applied, priced products per scrape run (184 runs): sugar median 97 (range 4–137; 6 runs below half the median); flour median 52 (range 8–71; 69 runs, 38%, below half the median).

## Failure patterns by category

| Pattern | Categories | Evidence |
|---|---|---|
| Unrelated listings pass the commodity filter; relevance falls with search depth | tea (severe), dairy (milder) | Name-term match 14.1% (tea) and 66.8% (dairy); manual search: tea relevant on pages 1–3, dairy on pages 1–2 |
| Generic keyword; most results removed by the filter | sugar, flour | Keywords `sugar`, `atta flour`; 75–90% removed, 6–9 priced items per run |
| Uneven coverage per scrape run | flour | 38% of runs below half the median priced items |
| Day-to-day product churn | oil | Median 12% of the previous day's priced products absent the next day; maximum 66% |


## Category status after month 4

| Category | Status | Basis |
|---|---|---|
| ghee | Included | Volatility 0.0158; noise ratio 1.38 (163 URLs); beat naive in 3 of 4 months |
| pulses | Included | Volatility 0.0287; noise ratio 1.27 (256 URLs); beat naive in 4 of 4 months |
| rice | Provisional | Volatility 0.0210; noise ratio 1.56 (103 URLs); missed naive in months 3 and 4 |
| sugar | Excluded | Volatility 0.2009; balanced set of 2 URLs at 80% presence; month-4 product-days added 367; passed naive in months 2–3 |
| dairy | Excluded | Missed naive in month 1; noise ratio readings 2.42 / 6.22 / 1.51 across thresholds; 28.2% of products without "Olper's" or a dairy term |
| oil | Excluded | Volatility 0.1162, 0.1304, 0.2577 in months 1–3; span ratio 17.99 → 34.72; daily product churn; month-4 export differs from month 3 |
| flour | Excluded | Volatility 0.0424 → 0.1552; balanced set under 30 URLs in every month; uneven coverage per run |
| tea | Excluded | 14.1% of product names contain a tea term; median days observed 3.0 in every month |

## Source files

- Panel, noise ratio, volatility, price spread and day/night gap: `panel_stats`, `stability_check`, `category_volatility_full`, `price_spread` and `day_night_median_gap` CSVs in `results/post-filter-update/cp1_1m/diagnostics/`, `results/post-filter-update/cp2_2m/diagnostics/`, `results/post-filter-update/cp3_3m/diagnostics/`, `results/post-filter-update/cp4_4m/diagnostics/`.
- Row counts and oil's price distribution: `price_spread` CSVs.
- Product-name relevance, dairy, oil, sugar and flour sections: raw `products_rows.csv` and `product_prices_rows.csv` (not in the repo).
- Tea: `results/investigations/tea_contamination/`.