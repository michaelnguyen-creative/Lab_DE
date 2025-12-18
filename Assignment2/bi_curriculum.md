# Business Intelligence Lab

In this learning path, we will treat each Project as a **Business Intelligence Deliverable**. Each project follows a standard professional lifecycle: Requirement Gathering, Data Logic (Pseudocode), and Strategic Insight.
- Dataset: AdventureWorks2025 (OLTP) with application to Amazon e-commerce domain

---

## Project 1: The Executive P&L Dashboard

**Project Goal:** Create a high-level "Command Center" (P/L statement) for the CFO to monitor the company’s financial health.
* Business Question: How is profitability trending? Why?

### Requirements:

* **Data Aggregation:** Consolidate raw sales transactions into a standardized Profit & Loss format.
* **Metric Definition:** Calculate Gross Profit and Gross Margin % at the annual level.
* **Comparison:** Must show year-over-year (YoY) growth in revenue.
* **Logic:** 
    * Find the difference between what we sold it for (LineTotal) and what it cost to make (COGS = OrderQty * StandardCost).
    * Aggregate by calendar year of the order date.



---

## Project 2: Product Catalog "Rationalization" Study

**Project Goal:** Audit the current product line to determine which items are driving growth and which are "dead weight."
* Business Question: What should we focus on? Discontinue? Double down on?

### Requirements:

* **Granularity:** Data must be viewable by Category, Subcategory, and Individual Product.
* **Volume Filter:** Exclude "noise" by filtering out products with fewer than 100 units sold.
* **Profitability Ranking:** Rank products by total net profit rather than gross revenue.
* **Logic:** 
    * Join the Sales tables with Product and Category tables.
    * Subtract the total cost (Unit Cost * Qty) from the revenue for every product.
    * Identify "Outliers" (Products with high revenue but negative profit).



---

## Project 3: Capital Efficiency & Inventory Audit

**Project Goal:** Identify "Cash Traps"—products that are sitting in the warehouse too long and costing the company money in storage.
* Business Question: Which products tie up too much cash? Which needs better inventory management?

### Requirements:

* **Turnover Calculation:** Determine how many times per year the inventory is fully replaced.
* **Risk Flagging:** Highlight products where current inventory levels are significantly higher than annual sales.
* **Logic:**
    * Get the average stock level from the Inventory snapshots.
    * Divide the annual quantity sold by that average stock level.
    * **IF** Turnover < 2, **THEN** flag as "Slow Moving/High Risk."



---

## Project 4: Customer Lifecycle & Segmentation (RFM)

**Project Goal:** Identify "Whales" (high spend/frequency) vs. "At-Risk" (high spend/long time since last purchase) => Move away from "one-size-fits-all" marketing by grouping customers based on their buying behavior.
* Business Question: Who are your best customers? How do you identify & retain them?

### Requirements:

* **Recency Metric:** Calculate the "Days Since Last Order" for every customer.
* **Monetary Metric:** Total lifetime value (LTV) per customer.
* **Frequency Metric:** Total number of distinct orders.
* **Logic:**
    * For each customer, find the MAX(OrderDate).
    * Count the number of SalesOrderIDs.
    * Sum the TotalDue.
* **Business Rule:** Customers with different Recency can be categoried as "At Risk, VIP, etc."



---

## Project 5: Demand Forecasting & Seasonality Map

**Project Goal:** Provide the Operations team with a roadmap for when to hire more staff and when to increase stock levels.
* Business Question: When should you stock up? Run promotions? Expect cash crunches?

### Requirements:

* **Temporal Analysis:** Group sales by Year and Month.
* **Seasonality Baseline:** Identify the 3 highest and 3 lowest months for sales volume.
* **Logic:**
    * Extract the Month and Year from the transaction timestamp.
    * Calculate the average revenue per month across multiple years to find "Peak Seasons."
    * Compare the Average Order Value (AOV) in December vs. July.



---

## Project 6: Price Elasticity & Revenue Optimization

**Project Goal:** Test if the company can increase prices without losing a disproportionate amount of sales volume.
* Business Question: What's the optimal price point? How sensitive are customers to price hikes?

### Requirements:

* **Price Point Correlation:** Map every unique price point offered for a specific product against the units sold at that price.
* **Revenue Curve:** Identify the "Price Ceiling" (where sales drop off to zero).
* **Logic:**
    * Group sales by Product and **Unit Price**.
    * Calculate Total Revenue for each price point.
* **Goal:** Find the price point where (Price * Quantity) is at its maximum.



---

## Summary of Your Portfolio

By the end of these 6 Projects, you will have built:

1. A **Financial Statement** (P&L)
2. A **Product ROI** Analysis
3. An **Inventory Velocity** Report
4. A **Customer Segmentation** Model
5. A **Seasonality Forecast**
6. A **Pricing Sensitivity** Study
