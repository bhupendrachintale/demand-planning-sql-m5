# Demand Planning Analytics in SQL: Walmart M5

Planning metrics that demand planners use every day in tools like SAP IBP (forecast accuracy, bias, ABC-XYZ segmentation, safety stock), rebuilt from raw point-of-sale data in SQL, plus a feature table ready for a machine-learning forecast.

**Why this project:** I've helped major organizations through their supply chain planning transformations, redesigning planning processes and implementing SAP IBP. Now I'm bringing AI into supply chain planning. I built these metrics myself, from raw data, so I understand the data, logic and pitfalls inside the AI planning tools I want to help companies adopt.

---

## Dataset
[Walmart M5 Forecasting](https://www.kaggle.com/competitions/m5-forecasting-accuracy) (Kaggle)
- 10 stores in California, Texas and Wisconsin
- 3,049 items across Foods, Household and Hobbies
- Daily unit sales from Jan 2011 to Jun 2016 (59 million rows), weekly prices, events and SNAP days

## Business questions and key findings

| # | Business question | Key finding |
|---|---|---|
| 01 | What does the data contain, and where are its traps? | 20.8% of rows have no price because the item wasn't on sale yet. Zero-sales days are 51–72% by category, so much of the catalog is intermittent demand. |
| 02 | How accurate is a simple baseline forecast by store? | A naive forecast (this week = last week) reaches 4–14% WMAPE at store level, with a small under-forecast bias in every store because sales grew over time. |
| 03 | Which items deserve the most planning attention? | AX items (high value, stable) are 37% of items but 68% of revenue, which makes them candidates for automated forecasting. AY/AZ items (6.5% of items, 12% of revenue) are where planner time pays off most. |
| 04 | How much safety stock does each item need? | Safety stock based on demand variability overstates risk for stable, trending items and understates it for intermittent ones. Some items show 50+ days of cover and should be reviewed, not applied automatically. |
| 05 | What would a machine-learning forecast need as inputs? | Built an item × week feature table (lags, rolling averages, price change, events, SNAP). Found **censored demand**: the top-selling item shows weeks of zero sales at an unchanged price, which points to stockouts that would mislead a model. |
| 07 | Which zero-sales days are real zero demand, and which are stockouts? | Grouped 28M zero-sales days into 6.6M runs and flagged runs too long to be chance for each item-store. About **29% of zero days are not real zero demand**: 7% look like stockouts (avg 18 days) and 22% like items not stocked in that store despite having a price (avg 135 days). |

## Repository structure
```
notebooks/
├── 00_setup_and_load.ipynb        Load CSVs, reshape, build the core tables
├── 01_data_profiling.ipynb        Calendar, prices, sales profiling
├── 02_forecast_accuracy.ipynb     Naive forecast, moving average, WMAPE, bias
├── 03_abc_xyz_segmentation.ipynb  ABC by revenue, XYZ by variability, matrix
├── 04_safety_stock.ipynb          Safety stock and days of cover by item
├── 05_ml_feature_table.ipynb      Feature table for an ML forecast (store CA_3)
├── 06_sql_recap_practice.ipynb    SQL practice questions and answers
└── 07_stockout_flag.ipynb         Stockout and inactive-item detection from sales patterns
requirements.txt
```

## Data model
| Table | One row = | Purpose |
|---|---|---|
| `sales_daily` | item × store × day | Main fact table: units, price, revenue |
| `weekly_store_sales` | store × week | Store forecasting and accuracy |
| `item_abc` / `item_xyz` | item | Value and variability classes |
| `v_abc_xyz_matrix` | segment | View combining both classes |
| `ml_features_ca3` | item × week (CA_3) | Model inputs |
| `stockout_runs_v2` | zero-sales run | Each run classified as normal, stockout or inactive |

## SQL techniques used
Joins at the correct grain (and proving when a join multiplies rows), CTEs, window functions (LAG, rolling averages, RANK, running totals, share of total), gaps and islands (finding consecutive-day runs), UNPIVOT, NULL handling, views, and data-quality checks after every table build.

## How to run
1. Download the M5 files from Kaggle into `data/raw/`.
2. Set up the environment:
   ```
   python -m venv .venv
   .venv\Scripts\Activate.ps1
   python -m pip install -r requirements.txt
   python -m jupyterlab
   ```
3. Run the notebooks in order (00 → 07), one at a time. Notebook 06 is optional practice.

**Tools:** Python 3.13, DuckDB, JupyterLab, pandas

## Limitations
- **No inventory data**, so stockouts can only be inferred from sales patterns. The thresholds used (1-in-1,000 chance, 56-day inactive boundary) are business assumptions that real on-hand and ranging data would validate.
- **No lead times**: safety stock assumes a 2-week lead time for all items.
- **No customer orders or live POS feeds**, which real demand sensing relies on.

## Next steps
- Use the stockout flag to correct demand history and recalculate safety stock
- Python and LightGBM forecast compared with the naive baseline
- Demand sensing on Favorita grocery data plus weather signals
- An AI planner assistant that explains forecast changes and flags stockout risk

## About me
Supply chain planning transformation professional. I've helped major organizations transform their supply chain planning through process redesign and SAP IBP implementations, and I'm now focused on applying AI to supply chain planning.
[LinkedIn](https://www.linkedin.com/in/bhupendra-chintale-7333482a/)
