# Hotel Bar Inventory Forecasting and Recommendation

## 1. Core Business Problem and End-to-End Solution Pipeline

Hotel bars need to maintain enough stock to meet customer demand without overstocking. Stockouts can lead to lost sales and dissatisfied guests, while excess inventory ties up cash and increases the risk of waste and breakage. The goal of this project was to forecast demand and recommend appropriate stock levels for each bar and brand.

**End-to-end pipeline implemented in `main.ipynb`:**

1. **Data loading and validation:** Loaded historical inventory records using Pandas, parsed timestamps, converted quantities to numeric values, and checked missing values, duplicates, negative quantities, and invalid timestamps. I also flagged discrepancies in inventory balances for review.

2. **Exploratory data analysis:** Used Pandas and Matplotlib to compare recorded consumption across bars and alcohol categories and understand historical demand patterns.

3. **Demand preparation:** Grouped consumption by bar, alcohol type, brand, and date. Multiple records from the same day were combined, and missing item-days were filled with zero recorded demand to create daily time series. Data coverage was also measured.

4. **Demand forecasting:** Compared an eight-week moving average with a previous-week baseline using chronological rolling validation and evaluated both with MAE, WAPE, and forecast bias.

5. **Inventory planning:** Used the selected forecast to calculate safety stock, reorder points, and par levels based on expected demand, supplier lead time, and review frequency.

6. **Inventory simulation:** Compared a fixed-stock baseline policy with the forecast-based policy, accounting for replenishment lead times, scheduled orders, and unmet demand.

7. **Recommendations and reporting:** Generated inventory recommendations for 96 bar-brand combinations, along with charts and a CSV output containing forecast, stock-target, and validity information.

## 2. Assumptions and Why I Made Them

- **Missing item-days were treated as zero recorded demand:** This created a consistent daily time series for forecasting and simulation. However, missing records could also indicate incomplete logging. Average recorded-day coverage was only 18.7%, making this an important limitation.
- **Supplier lead time was set to 3 days, with a 7-day review interval:** Actual supplier timing and review schedules were unavailable, so these were configurable planning assumptions.
- **Service level was set to 95%:** This provided a target for calculating safety stock.
- **Stock quantities were rounded up to 50 ml:** This established a consistent planning increment because bottle and case pack sizes were unavailable.
- **Unmet demand was treated as lost sales:** The data did not indicate whether customers would wait or choose alternative drinks.
- **Both policies used the same modeled opening stock:** A complete opening-stock snapshot was unavailable, so a common starting-stock assumption was used for a fair comparison.
- **Historical stock balances were not considered current:** The dataset ended on January 1, 2024. Only 17 inventory positions passed the notebook's validity checks, and these still require current stock verification.

## 3. Model Selection and Why I Chose It

I compared two simple forecasting methods:

- **Eight-week moving average:** Averages demand over the previous eight complete weeks, helping smooth short-term fluctuations.
- **Previous-week baseline:** Uses the previous complete week's demand as the forecast.

Using chronological rolling validation across 768 item-weeks, the eight-week moving average achieved an MAE of **325.1 ml** and WAPE of **85.4%**. The previous-week baseline achieved an MAE of **389.5 ml** and WAPE of **102.3%**.

I selected the eight-week moving average because it produced lower forecasting errors on both metrics. Although the previous-week baseline had slightly lower bias, its overall error was higher.

I did not use more complex machine-learning or seasonal models because the dataset covered approximately one year and contained sparse records, with around 68 recorded days per bar-brand combination on average. Better data coverage was a higher priority than model complexity.

## 4. System Performance and Improvements

The historical simulation covered 253 days across 96 bar-brand combinations. The recommended policy reduced simulated stockout days from 1,838 to 375, a decrease of approximately 79.6%, and reduced unmet demand from 467,927.2 ml to 85,360.2 ml, an improvement of approximately 81.8%. However, average inventory increased from 33,412.2 ml to 93,590.4 ml, and total ordered volume rose from 865,384.4 ml to 1,324,816.4 ml. This highlights a trade-off between product availability and inventory investment rather than proving financial savings. The main limitation was incomplete historical data, with only 18.7% average recorded-day coverage. To improve the system, I would collect more complete daily consumption and stock records, distinguish genuine zero-demand days from missing records, incorporate actual supplier lead times and purchase pack sizes, and evaluate performance using real inventory, waste, and stockout costs. With better data, I would also test seasonal and weekday demand patterns to improve forecasting accuracy.

## 5. How It Would Work in a Real Hotel

A hotel could integrate its point-of-sale and inventory systems to supply daily consumption, purchases, current stock, and outstanding orders for each bar and brand.

At each scheduled review, the system would update the demand forecast, calculate the reorder point and par level, and check the inventory position, including open orders. If replenishment were required, it would recommend an order quantity and present it to the manager for verification and approval.

Once deliveries arrived, actual consumption, stock counts, wastage, and order adjustments would feed back into the system to improve future forecasts and inventory decisions.

## 6. What Could Break at Scale and What to Monitor

At scale, the current notebook-based workflow could struggle with large volumes of inventory records, incomplete data feeds, duplicate imports, inconsistent bar or brand identifiers, stale stock counts, and different supplier policies. Generating a complete daily grid for thousands of items could also increase memory usage.

A production system would need reliable data ingestion, stable item and location identifiers, configurable supplier settings, scheduled processing, audit logs, access controls, and alerts for invalid or stale data.

I would monitor:

-**Data quality:** Missing records, data coverage, duplicate entries, and inventory balance discrepancies.
- **Forecast accuracy:** MAE, WAPE, and forecast bias by bar and brand.
- **Inventory performance:** Stockout days, unmet demand, and fill rate.
- **Inventory efficiency:** Average inventory, replenishment volume, turnover, and waste or expiry.
- **Operational reliability:** Actual supplier lead times, delayed deliveries, stale stock updates, and manager overrides.

These metrics should be reviewed at the individual bar and brand level, since chain-wide averages can hide local problems.
