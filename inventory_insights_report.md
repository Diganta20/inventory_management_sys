# Inventory demand forecasting and par-level planning

## Executive summary

The supplied file has 6,575 movement records for six bars and 16 brands, covering 1 January 2023 through 1 January 2024. It supports a first simulation of item-level planning, but it is not a complete daily demand ledger: the 96 bar-brand combinations have about 68 recorded days each on average. A missing row may mean no logged activity or a missed record.

The notebook builds daily series, compares an eight-week moving-average forecast with a previous-week baseline on a rolling eight-week holdout, calculates a par target with a safety buffer, and exports a manager-ready CSV. On 768 rolling item-weeks, the moving average scored 83.8% WAPE and 327.7 ml MAE, versus 104.4% WAPE and 408.2 ml MAE for the previous-week baseline. The forecast is better than this simple benchmark, but the error is still high; treat it as a starting point pending a stock count and logging-quality review.

## What the data says

- Recorded consumption is about 1.97 million ml over the date range.
- Vodka has the largest recorded total by alcohol type (about 412,000 ml); wine is lowest (about 379,000 ml). Differences are modest, so local bar-brand combinations matter more than broad category labels.
- Johnson's Bar has the largest recorded total among the six bars (about 344,000 ml); Taylor's Bar has the smallest (about 315,000 ml).
- The original fields have no null values and there are no exact duplicate rows, but recording is sparse: each bar-brand combination appears on 40 to 99 days of the 365-day period.
- The inventory balance equation is generally consistent with opening balance plus purchases minus consumption equaling closing balance, though a few rows have larger differences. Reconcile these before trusting balances operationally.

These are recorded totals, not normalized sales rates; differences in logging frequency can affect comparisons.

## Forecast and inventory logic

1. Aggregate rows for a bar, brand, and calendar day.
2. Complete the daily calendar and treat absent rows as zero recorded consumption. Display observed-day coverage alongside results.
3. Forecast the next four weeks with average weekly consumption from the latest eight weeks. Evaluate rolling predictions on the final eight weeks and report MAE, WAPE, and bias against the previous-week baseline.
4. Set par to expected use over lead time plus review period, plus safety stock: average daily demand multiplied by protection days, plus a service-level z-score multiplied by daily standard deviation and the square root of protection days.
5. Calculate suggested order as the positive difference between par and the last recorded closing balance. That balance is historical, not current stock.

## Assumptions and limits

- Starter settings are a three-day replenishment lead time, seven-day review cycle, and 95% service level. Managers should replace these with supplier- and location-specific values.
- Zero-filling missing dates can understate demand if records are incomplete. This is the main risk to forecast and par quantities. Coverage is included to direct record checks.
- The latest data is from 1 January 2024. Refresh stock before ordering.
- The eight-week moving average is explainable and suitable as a first baseline for short, sparse series. One year of data is not enough to establish dependable annual seasonality.
- Recommendations remain in millilitres. Apply pack sizes, minimum order quantities, expiry, and local operating constraints before purchasing.

## Suggested operating workflow

Refresh the input from a complete consumption and stock ledger. Review coverage and the forecast scorecard, then check live balances for below-par or stale records. Convert the target to purchasable packs, approve replenishment, and compare stockouts and waste with recommendations weekly. With several months of complete data, add weekday and event effects and tune service levels by item importance.

## Deliverables

- main.ipynb ? documented analysis, rolling validation, forecasts, par targets, and export.
- inventory_recommendations.csv ? generated when the notebook is run; includes coverage, last stock timestamp, and planning assumptions.
