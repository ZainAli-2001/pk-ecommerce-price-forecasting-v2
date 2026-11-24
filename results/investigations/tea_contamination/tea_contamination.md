# Tea Contamination

Tea is excluded from modelling because most listings in its panel are not tea. This file documents the evidence and the two exported lists in this folder.

## Files

Both files are written by `notebooks/category_diagnostics.ipynb` from the products table, after the commodity filter (`is_valid_commodity` from the pipeline repository's `utils/commodity_filters.py`):

- `tea_kept.csv`: tea products that passed the filter. The tea panel in the diagnostics tables is built from these. 1,148 rows.
- `tea_excluded.csv`: tea products removed by the filter. 1,968 rows.
- Columns: an unnamed index (the row index in the products table, not the product `id`) and `name`.
- The lists are the latest export and are not cleaned, apart from the redaction below.

## Redaction

11 rows (10 in `tea_kept.csv`, 1 in `tea_excluded.csv`) have the name replaced with `[REDACTED - explicit product]`. The rows are retained, so row counts match the original exports. The redacted listings are adult products: in the kept list 2 explicit product listings, 5 listings of one brand whose name contains "danedar" (the word in the search keyword), 2 listings with deliberately misspelled titles and 1 tablet listing; in the excluded list 1 explicit product listing.

## Name relevance

A name is counted as relevant if it contains at least one of: tea, chai, danedar, tapal, lipton, patti, chaye, karak, tezdum, family mixture, dahleez, bengal, zordar, harmony, yellow label. Redacted rows are counted separately.

| Source | Rows | Contain a tea term | No tea term | Redacted |
|---|---|---|---|---|
| Raw products table, tea (not in the repo) | 3,116 | 13.7% | 85.9% | 0.35% |
| `tea_kept.csv` | 1,148 | 319 (27.8%) | 819 (71.3%) | 10 |
| `tea_excluded.csv` | 1,968 | 109 (5.5%) | 1,858 (94.4%) | 1 |
| Both files | 3,116 | 428 (13.7%) | 2,677 (85.9%) | 11 |

Of the 109 excluded rows that contain a tea term, 79 contain tea-bag, premix/instant or `T/B` wording (68 contain "bag" or "bags", 6 contain premix, "3 in 1" or instant, 5 contain `T/B`), which are the filter's intended exclusions. The other 30 do not.

## Unrelated listings

Examples seen in both lists: books and novels (Urdu and English), prayer beads, caps and pouches, rings and jewellery, cartoon and anime merchandise, party and cake decorations, dice and board games, stationery, clothing, electronics accessories, furniture, and adult products (redacted).

## Search behaviour

The Daraz keyword for tea is `tapal danedar`. Manual search on the Daraz site: pages 1–3 show relevant tea products; from page 4 the results are unrelated. Other keywords tried: `chai` returns specialty and herbal teas and other non-commodity items; `lipton` and other brand names return about one page of relevant results each.

## Why the commodity filter does not remove them

The tea filter is a list of exclusion phrases and patterns (tea bags, premix, coffee, latte/cappuccino, the `T/B` abbreviation). A listing unrelated to tea contains none of them and passes. Titles that are deliberately misspelled do not match the phrases either.

## Effect on the tea panel

Cumulative panel statistics from the diagnostics outputs:

| Month | Products | % with no price variation | Median days observed per product |
|---|---|---|---|
| 1 | 705 | 89.2% | 3.0 |
| 2 | 893 | 88.0% | 3.0 |
| 3 | 1,021 | 88.4% | 3.0 |
| 4 | 1,096 | 88.4% | 3.0 |

The product count grows each month while the median days observed stays at 3.0.

## Status

Tea is excluded from baseline modelling. No keyword has been found that returns tea listings without a large share of unrelated ones.