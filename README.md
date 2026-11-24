# pk-ecommerce-price-forecasting-v2

Category-level price diagnostics and baseline forecasts for Daraz grocery staples (rice, ghee, pulses, sugar, dairy, oil, flour, tea), built on data from pipeline version 2.0.0 of [pk-ecommerce-price-pipeline](https://github.com/ZainAli-2001/pk-ecommerce-price-pipeline). "v2" refers to the pipeline version, not to a second iteration of this repository.

## Scope

- Source: Daraz only.
- Data: pipeline 2.0.0 rows, 2026-07-16 to 2026-11-15 (four months).
- Grain: daily median `unit_price` per category (PKR per kg or per litre).
- Per-product forecasting is not part of this repository.
- Earlier work on pipeline 1.0.0 data, which scoped per-product modelling to tea, is in `pk-ecommerce-price-analysis`. Under pipeline 2.0.0 tea is excluded (`results/investigations/tea_contamination/tea_contamination.md`).

## Status after month 4

| Category | Status | Basis |
|---|---|---|
| ghee | Included | Beat naive persistence in 3 of 4 months; volatility 0.0158 |
| pulses | Included | Beat naive persistence in 4 of 4 months (month-4 margin 1.3%); volatility 0.0287 |
| rice | Provisional | Beat naive in months 1–2, missed in months 3 and 4 |
| sugar | Excluded | Daily-median volatility 0.1143–0.2009; balanced panel of 2 URLs at month 4 |
| dairy | Excluded | Missed naive in month 1; 28.2% of products contain neither "Olper's" nor a dairy term |
| oil | Excluded | Volatility 0.1162, 0.1304, 0.2577 in months 1–3; day-to-day product churn |
| flour | Excluded | Volatility 0.0424 → 0.1552; balanced set under 30 URLs in every month |
| tea | Excluded | 14.1% of product names contain a tea term; median days observed 3.0 |

Full evidence is in `docs/post-filter-update/`.

## Repository layout

```
notebooks/
  category_diagnostics.ipynb                      diagnostics for all categories
  data_preprocessing_and_baseline_model.ipynb     daily category series and baseline evaluation
  pre-filter-update/
    forecasting_stage2.ipynb                      session and daily files, store comparison, day/night checks, spike flags
    feature_engineering_gated.ipynb               gated features for ghee, rice and tea
docs/
  pre-filter-update/
    1m_jul16-aug16/                               three documents, month 1
    2m_jul16-sep15/                               three documents, month 2
  post-filter-update/
    baseline_model_months_1-3_comparison__...md
    category_diagnostics_months_1-3_comparison__...md
    baseline_model_months_1-4_comparison__...md
    category_diagnostics_months_1-4_comparison__...md
results/
  pre-filter-update/
    cp1_1m_jul16-aug16/baseline/
    cp2_2m_jul16-sep15/baseline/
  post-filter-update/
    cp1_1m/ cp2_2m/ cp3_3m/ cp4_4m/               each with diagnostics/ and baseline/
  investigations/
    tea_contamination/                            tea_contamination.md, tea_kept.csv, tea_excluded.csv
data/
  raw/                                            not in the repository
vendor/
  pipeline/                                       sparse checkout of utils/ from the pipeline repository
```

## Documents

- `docs/post-filter-update/`: the analysis under the current commodity filter. The months 1–3 pair covers data through 2026-10-15. The months 1–4 pair adds month 4 and the investigations done on the raw data (dairy, oil, sugar and flour). Each baseline document has a diagnostics companion.
- `docs/pre-filter-update/`: the first analysis, under the original commodity filter, at months 1 and 2 (three documents each). The pipeline repository's commodity filter was changed during version 2.0.0 without a version bump; "pre-filter-update" and "post-filter-update" name the two filter states, and numbers are not comparable across them. Folder and file names use months; the text inside these documents uses day counts. Each document lists its sources: the baseline tables are backed by `results/pre-filter-update/`, some figures by saved outputs of the notebooks in `notebooks/pre-filter-update/`, and the remaining figures have no supporting file in this repository. Their conclusions are not carried into the post-filter-update documents.

## Results

- A checkpoint is a month-end. Each folder holds the outputs of one run on the cumulative window up to that month-end.
- `diagnostics/` holds the CSVs written by `category_diagnostics.ipynb`: panel statistics, stability check, scorecard, category volatility (full window and three windows), price spread and the day/night median gap.
- `baseline/` holds `baseline_model_results.csv` and `category_daily_baseline_features.csv` from `data_preprocessing_and_baseline_model.ipynb`.
- The month-1 to month-3 diagnostics were run on 2026-10-18 and the month-4 diagnostics on 2026-11-20. The notebooks import `utils/` from the live pipeline repository, so each run used the commodity filter current on its run date.
- `day_night_paired_*.csv` (0.6–2.3 MB each) is written by the diagnostics notebook and is excluded by `.gitignore`.

## Notebooks

- `category_diagnostics.ipynb` reads `data/raw/products_rows.csv` and `data/raw/product_prices_rows.csv`, keeps `pipeline_version` 2.0.0, applies `is_valid_commodity`, collapses the morning and evening sessions to one row per product per day, and writes its outputs under `data/output/diagnostics_output/`. It also writes the tea product lists.
- `data_preprocessing_and_baseline_model.ipynb` runs the same preparation, builds the category daily series with lag, rolling and discount features, and evaluates the model against naive persistence, a 7-day rolling mean and a discount-aware naive baseline for the categories in `CLEARED_CATEGORIES` (currently rice, ghee, pulses). Outputs go under `data/output/forecast_data/`.
- `notebooks/pre-filter-update/` holds the three notebooks used before the two above. Their saved outputs are from specific runs: `forecasting_stage2.ipynb` holds the post-filter month-1 run (its day/night counts and 32,738 paired product-days equal the `cp1_1m` files), `feature_engineering_gated.ipynb` holds the pre-filter month-2 run (88,611 daily rows), and `diagnostic_analysis.ipynb` inspects the highest and lowest unit prices for dairy, flour and sugar in the recalculated data from before 2026-07-16.
- Both notebooks above write to `data/output/`, which is excluded by `.gitignore`; their files were copied into `results/` under checkpoint names. The notebooks reflect the month-4 run. Earlier checkpoints were produced by earlier versions of the notebooks; the evaluated categories were rice, ghee, pulses and dairy at month 1, and rice, ghee, pulses and sugar at months 2 and 3.

## Setup

`utils/` comes from the pipeline repository through a sparse-checkout submodule:

```bash
git submodule add https://github.com/ZainAli-2001/pk-ecommerce-price-pipeline vendor/pipeline
cd vendor/pipeline
git sparse-checkout init --cone
git sparse-checkout set utils
cd ../..
```

On a fresh clone:

```bash
git submodule update --init --recursive
pip install -r requirements.txt
```

## Data availability

The raw data (`products_rows.csv`, `product_prices_rows.csv`) is not in this repository. It is large, and I kept it out to keep the repository small. If you want the data, contact me through my GitHub profile (ZainAli-2001). Figures in the documents that come from the raw tables are marked "not in the repo".

## Related repositories

- [pk-ecommerce-price-pipeline](https://github.com/ZainAli-2001/pk-ecommerce-price-pipeline): scrapers, database layer and `utils/`.
- `pk-ecommerce-price-analysis`: analysis on pipeline 1.0.0 data.