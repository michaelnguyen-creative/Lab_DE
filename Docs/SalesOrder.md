## 📘 AdventureWorks Sales Order Documentation
### Order Value Analytics Reference Guide

---

## 1. Overview & Business Context

### **1.1 Purpose of This Documentation**
This document provides business domain knowledge for AdventureWorks sales order data to support:
- SQL query development for order value analytics
- Feature engineering for ML/DL models
- Understanding business rules and metrics
- Ensuring correct interpretation of sales data

### **1.2 AdventureWorks Business Model**
AdventureWorks is a fictional bicycle manufacturer and retailer that:
- Sells products directly to consumers (online/retail)
- Sells products through reseller partners (wholesale)
- Operates across multiple geographic territories
- Maintains product catalog organized by category hierarchies
- Tracks complete order lifecycle from placement to shipment

### **1.3 Scope**
This documentation covers **OLTP schema only** (transactional database):
- Focus: Order-level and line-item-level data
- Core entities: Order, Customer, Product
- Analytical goal: Understanding order value patterns

---

## 2. Conceptual Data Model

### **2.1 Header-Detail Pattern**

**Business Concept:**
One customer order can contain multiple products. The order has:
- **Header**: Overall order information (who, when, total cost)
- **Details**: Individual product lines within that order (what, how many, unit price)

**Relationship:**
```
One Order (Header)
    ├── Contains: Multiple Line Items (Details)
    ├── Placed by: One Customer
    ├── Ships to: One Address
    └── Results in: One Total Amount Due
```

**Example:**
- Customer John Smith places **one order** on May 31, 2011
- That order contains **four different bikes** (four line items)
- The order total is $23,153.69
- Each line item contributes to that total

---

### **2.2 Core Entities**

#### **Order (Sales.SalesOrderHeader)**
**Definition:** A single purchase transaction from a customer

**Grain:** One row = one complete order

**Business Attributes:**
- **Who**: Customer who placed the order
- **When**: Order date, due date, ship date
- **Where**: Territory, shipping address
- **How Much**: Total amount due (including tax and freight)
- **Status**: Order processing status

---

#### **Line Item (Sales.SalesOrderDetail)**
**Definition:** A single product within an order

**Grain:** One row = one product in one order

**Business Attributes:**
- **What**: Specific product ordered
- **How Many**: Quantity ordered
- **Unit Economics**: Price per unit, discount applied
- **Line Total**: Quantity × Price × (1 - Discount)

---

#### **Customer (Sales.Customer)**
**Definition:** Individual or organization that purchases products

**Grain:** One row = one unique customer

**Business Attributes:**
- **Type**: Individual consumer vs. store/reseller
- **Territory**: Geographic sales region
- **Person Link**: Individual customer demographics (if applicable)

---

#### **Product (Production.Product)**
**Definition:** Sellable item in the catalog

**Grain:** One row = one unique product SKU

**Business Attributes:**
- **Category Hierarchy**: Category → Subcategory → Product
- **Pricing**: List price, standard cost
- **Physical Attributes**: Color, size, weight
- **Lifecycle**: Make vs. buy, discontinued status

---

## 3. Relationship Navigation Paths

### **3.1 Order → Customer Analysis**
**Business Question Pattern:** "Who are our customers and what do they buy?"

**Navigation:**
```
Order (SalesOrderHeader)
    → via CustomerID
Customer (Sales.Customer)
    → via PersonID (for individuals)
Person (Person.Person)
    
    → via TerritoryID (from Customer)
Territory (Sales.SalesTerritory)
```

**Key Dimensions:**
- Customer segmentation (individual vs. business)
- Geographic analysis (territory, region, country)
- Customer demographics (if individual person)

---

### **3.2 Order → Product Analysis**
**Business Question Pattern:** "What products drive revenue?"

**Navigation:**
```
Order (SalesOrderHeader)
    → via SalesOrderID
Line Item (SalesOrderDetail)
    → via ProductID
Product (Production.Product)
    → via ProductSubcategoryID
Subcategory (Production.ProductSubcategory)
    → via ProductCategoryID
Category (Production.ProductCategory)
```

**Key Dimensions:**
- Product category hierarchy (3 levels)
- Product attributes (color, size)
- Cost structure (list price vs. standard cost)

---

### **3.3 Temporal Analysis**
**Business Question Pattern:** "How do sales patterns change over time?"

**Key Date Fields:**
- **OrderDate**: When customer placed order (primary for time-series)
- **DueDate**: When customer expects delivery
- **ShipDate**: When order actually shipped

**Time Hierarchies:**
```
OrderDate
    → Year
        → Quarter
            → Month
                → Week
                    → Day of Week
```

---

## 4. Order Value Metrics: Definitions & Calculations

### **4.1 Line-Item Level Metrics**

#### **LineTotal**
- **Business Definition:** Total revenue for one product in one order
- **Calculation Logic:** `OrderQty × UnitPrice × (1 - UnitPriceDiscount)`
- **Interpretation:** How much this specific product contributes to order revenue
- **Excludes:** Tax and freight (those are order-level)
- **ML Feature Use:** Product-level revenue contribution, discount sensitivity

---

### **4.2 Order-Level Metrics**

#### **SubTotal**
- **Business Definition:** Sum of all line item totals before tax and freight
- **Calculation Logic:** `SUM(LineTotal)` across all line items in order
- **Interpretation:** Pure product revenue before additional charges
- **Business Rule:** SubTotal = sum of all LineTotals for that SalesOrderID
- **ML Feature Use:** Base order value, product mix total

#### **TaxAmt**
- **Business Definition:** Sales tax charged on the order
- **Calculation Logic:** Territory-specific tax rate applied to SubTotal
- **Interpretation:** Government-mandated charge, varies by location
- **Business Rule:** Calculated based on ship-to territory tax regulations
- **ML Feature Use:** Geographic tax burden, compliance costs

#### **Freight**
- **Business Definition:** Shipping and handling charges
- **Calculation Logic:** Based on weight, distance, shipping method
- **Interpretation:** Logistics cost passed to customer
- **Business Rule:** May be waived for high-value orders or promotions
- **ML Feature Use:** Delivery cost component, geographic distance proxy

#### **TotalDue** ⭐ **PRIMARY ORDER VALUE METRIC**
- **Business Definition:** Total amount customer must pay
- **Calculation Logic:** `SubTotal + TaxAmt + Freight`
- **Interpretation:** Complete order value from customer perspective
- **Business Rule:** This is the "order value" for most analytics
- **ML Feature Use:** Target variable for order value prediction, segmentation

---

### **4.3 Derived Metrics for Analytics**

#### **Average Order Value (AOV)**
- **Business Definition:** Mean order value across a set of orders
- **Calculation Logic:** `SUM(TotalDue) / COUNT(DISTINCT SalesOrderID)`
- **Granularity:** Calculated at segment level (customer, territory, time period)
- **Interpretation:** Typical order size for a group
- **Use Case:** Customer segmentation, channel comparison, trend analysis

#### **Order Line Count**
- **Business Definition:** Number of distinct products in an order
- **Calculation Logic:** `COUNT(DISTINCT ProductID)` per SalesOrderID
- **Interpretation:** Order complexity, basket size
- **Business Insight:** High-value orders may contain more products
- **Use Case:** Basket analysis, cross-sell effectiveness

#### **Product Mix Diversity**
- **Business Definition:** Category variety within an order
- **Calculation Logic:** `COUNT(DISTINCT CategoryID)` per SalesOrderID
- **Interpretation:** Cross-category purchasing behavior
- **Business Insight:** Single-category vs. multi-category shoppers
- **Use Case:** Merchandising strategy, customer profiling

#### **Discount Rate**
- **Business Definition:** Percentage discount applied to products
- **Calculation Logic:** `UnitPriceDiscount` field (0.00 to 1.00)
- **Interpretation:** Price concession to customer
- **Business Rule:** 0.00 = full price, 0.15 = 15% off
- **Use Case:** Promotional effectiveness, margin analysis

#### **Unit Economics**
- **Business Definition:** Profitability per product
- **Calculation Logic:** `(UnitPrice - StandardCost) × OrderQty`
- **Interpretation:** Gross profit contribution per line item
- **Requires:** StandardCost from Product table
- **Use Case:** Product profitability, portfolio optimization

---

## 5. Business Rules & Data Quality Considerations

### **5.1 Order Status Codes**

Orders progress through lifecycle stages:
- **Status = 1**: In process (not yet shipped)
- **Status = 2**: Approved (ready for fulfillment)
- **Status = 3**: Backordered (awaiting inventory)
- **Status = 4**: Rejected (customer issue, payment failed)
- **Status = 5**: Shipped (completed)
- **Status = 6**: Cancelled (customer or business initiated)

**Analytical Impact:**
- **Include in revenue analysis**: Status = 5 (Shipped) only
- **Exclude from metrics**: Status = 4 (Rejected) or 6 (Cancelled)
- **Pipeline analysis**: All statuses except 4 and 6
- **Churn analysis**: Consider cancelled orders (Status = 6)

---

### **5.2 Discount Application Logic**

**Business Rules:**
- Discounts are applied at **line-item level**, not order level
- `UnitPriceDiscount` ranges from 0.00 (no discount) to 1.00 (100% off, rare)
- Multiple products in same order can have different discount rates
- Discounts may come from: promotions, bulk purchases, special offers
- After discount is applied: `ActualPrice = UnitPrice × (1 - UnitPriceDiscount)`

**Edge Cases:**
- Some orders have mix of discounted and full-price items
- Discount = 0.00 is most common (majority of line items)
- High discounts (>30%) may indicate clearance, returns, employee purchases

---

### **5.3 Order Value Calculation Rules**

**Guaranteed Relationships:**
```
1. LineTotal = OrderQty × UnitPrice × (1 - UnitPriceDiscount)
2. SubTotal = SUM(LineTotal) for all line items in order
3. TotalDue = SubTotal + TaxAmt + Freight
```

**Data Quality Checks:**
- SubTotal should equal sum of LineTotal (within rounding tolerance)
- TotalDue should always be ≥ SubTotal (tax and freight are additive)
- OrderQty should always be positive integer
- UnitPrice should always be positive

---

### **5.4 Time-Based Business Rules**

**Date Relationships:**
```
OrderDate ≤ DueDate ≤ ShipDate (expected order)
```

**Business Logic:**
- **OrderDate**: Transaction timestamp, used for revenue recognition
- **DueDate**: Customer expectation, may drive operational metrics
- **ShipDate**: Actual fulfillment, used for delivery performance

**Analytical Considerations:**
- Use **OrderDate** for time-series revenue analysis (when sale occurred)
- Use **ShipDate** for operational efficiency metrics (fulfillment speed)
- **DueDate - OrderDate** = promised lead time
- **ShipDate - OrderDate** = actual lead time (delivery performance)

---

### **5.5 Currency & Internationalization**

**Assumption for Most Analyses:**
- All prices in USD (US Dollar)
- TaxAmt calculated in same currency
- No currency conversion needed for single-territory analysis

**Note for Multi-Territory:**
- If analyzing across countries, verify currency consistency
- Exchange rates may be stored separately if applicable

---

## 6. Analytical Patterns & Use Cases

### **6.1 Order-Level Analysis**
**Business Question:** "What's our average order value?"

**Conceptual Approach:**
- Treat each order as one observation
- Primary metric: TotalDue (complete order value)
- Group by: Customer, Territory, Time Period
- Filter: Exclude cancelled/rejected orders (Status 4, 6)

**What You're Measuring:**
- Customer purchasing behavior (order size)
- Geographic market differences
- Temporal trends (seasonality, growth)

---

### **6.2 Product-Level Analysis**
**Business Question:** "Which products drive the most revenue?"

**Conceptual Approach:**
- Treat each line item as one observation
- Primary metric: LineTotal (product contribution to revenue)
- Group by: Product, Category, Subcategory
- Aggregate: Sum LineTotal across all orders

**What You're Measuring:**
- Product portfolio performance
- Category contribution to revenue
- Product mix in orders

---

### **6.3 Customer Segmentation**
**Business Question:** "Who are high-value vs. low-value customers?"

**Conceptual Approach:**
- Aggregate orders per customer
- Calculate: Total lifetime value (sum of TotalDue)
- Calculate: Average order value (mean of TotalDue)
- Calculate: Order frequency (count of orders)

**Segmentation Dimensions:**
- **Monetary**: Total/average spend
- **Frequency**: Number of orders
- **Recency**: Days since last order
- **Geography**: Territory
- **Product Affinity**: Preferred categories

---

### **6.4 Time-Series Analysis**
**Business Question:** "How do sales patterns change over time?"

**Conceptual Approach:**
- Aggregate orders by time period (day/month/quarter)
- Calculate: Total revenue (sum TotalDue)
- Calculate: Order count
- Calculate: Average order value

**Patterns to Detect:**
- Seasonality (monthly/quarterly cycles)
- Growth trends (YoY, MoM)
- Day-of-week effects
- Holiday impact

---

### **6.5 Basket Analysis**
**Business Question:** "What products are purchased together?"

**Conceptual Approach:**
- Group line items by SalesOrderID (order as "basket")
- Identify co-occurring products
- Calculate: Order value when products purchased together
- Measure: Lift, support, confidence (association rules)

**Business Value:**
- Cross-sell opportunities
- Bundle pricing strategies
- Product placement optimization

---

## 7. ML/DL Feature Engineering Guidance

### **7.1 Target Variables (Prediction Tasks)**

#### **Order Value Prediction**
- **Target:** TotalDue (continuous regression)
- **Business Goal:** Forecast future order values
- **Use Case:** Inventory planning, revenue forecasting

#### **Customer Segmentation**
- **Target:** Customer lifetime value category (classification)
- **Business Goal:** Identify high-value customer segments
- **Use Case:** Marketing prioritization, retention strategy

#### **Churn Prediction**
- **Target:** Binary (will customer return?)
- **Business Goal:** Proactive retention
- **Use Case:** Loyalty programs, win-back campaigns

---

### **7.2 Feature Categories**

#### **Order-Level Features**
- SubTotal, TaxAmt, Freight, TotalDue
- Order status
- Lead time (DueDate - OrderDate)
- Delivery performance (ShipDate - OrderDate)

#### **Line-Item Features (Aggregated to Order)**
- Number of line items (order complexity)
- Number of unique products
- Number of unique categories (diversity)
- Average discount rate across line items
- Max/min line total (item value range)

#### **Customer Historical Features**
- Total lifetime orders (count)
- Total lifetime value (sum TotalDue)
- Average order value
- Order frequency (orders per month)
- Recency (days since last order)
- Tenure (days since first order)

#### **Product Features**
- Product category (categorical)
- Price point tier (budget/mid/premium)
- Historical sales volume
- Average discount rate for product
- Profit margin (if StandardCost available)

#### **Temporal Features**
- Order year, quarter, month, day of week
- Days since previous order (customer-level)
- Rolling averages (e.g., 30-day moving average order value)
- Seasonal indicators (holiday periods)

#### **Geographic Features**
- Territory (categorical)
- Region/Country (hierarchical)
- Territory-level aggregates (market size, average order value)

---

### **7.3 Feature Engineering Patterns**

#### **Recency-Frequency-Monetary (RFM)**
```
Pseudo-code logic:
For each customer:
    Recency = days_between(today, max(OrderDate))
    Frequency = count(distinct SalesOrderID)
    Monetary = sum(TotalDue)
    
    RFM_Score = weighted_combination(R_percentile, F_percentile, M_percentile)
```

#### **Customer Lifetime Value (CLV)**
```
Pseudo-code logic:
For each customer:
    Total_Revenue = sum(TotalDue where Status = 5)
    Total_Orders = count(distinct SalesOrderID where Status = 5)
    Avg_Order_Value = Total_Revenue / Total_Orders
    Order_Frequency = Total_Orders / tenure_in_months
    
    CLV = Avg_Order_Value × Order_Frequency × expected_lifetime_months
```

#### **Order Complexity Index**
```
Pseudo-code logic:
For each order:
    Line_Count = count(distinct line items)
    Category_Diversity = count(distinct categories)
    Price_Range = max(UnitPrice) - min(UnitPrice)
    
    Complexity_Score = normalized(Line_Count + Category_Diversity + Price_Range)
```

#### **Discount Sensitivity**
```
Pseudo-code logic:
For each customer:
    Orders_With_Discount = count(orders where any line item has discount > 0)
    Total_Orders = count(all orders)
    
    Discount_Affinity = Orders_With_Discount / Total_Orders
```

---

## 8. Common Analytical Mistakes to Avoid

### **8.1 Aggregation Level Confusion**
❌ **Wrong:** Averaging LineTotal to get average order value
```
Problem: LineTotal is per product, not per order
Result: Understates true average order value
```

✅ **Correct:** Average TotalDue from SalesOrderHeader
```
Reason: TotalDue represents complete order value
```

---

### **8.2 Including Cancelled Orders**
❌ **Wrong:** Including all orders regardless of status
```
Problem: Status = 4 (Rejected) or 6 (Cancelled) are not completed sales
Result: Overstates revenue and order count
```

✅ **Correct:** Filter for Status = 5 (Shipped) only
```
Reason: Only shipped orders represent realized revenue
```

---

### **8.3 Double-Counting Revenue**
❌ **Wrong:** Summing both LineTotal (from Detail) and TotalDue (from Header)
```
Problem: TotalDue already includes all LineTotals
Result: Revenue counted twice
```

✅ **Correct:** Choose appropriate grain
```
- Product analysis → Sum LineTotal
- Order analysis → Sum TotalDue
- Never sum both
```

---

### **8.4 Ignoring Header-Detail Relationship**
❌ **Wrong:** Analyzing line items without joining to header
```
Problem: Missing order-level context (customer, date, status)
Result: Cannot link product sales to customer or time dimensions
```

✅ **Correct:** Join Detail to Header for dimensional analysis
```
Reason: Header contains customer, date, status information
```

---

### **8.5 Time-Based Attribution Errors**
❌ **Wrong:** Using ShipDate for revenue recognition
```
Problem: Revenue should be recognized when order is placed, not shipped
Result: Misaligned financial reporting
```

✅ **Correct:** Use OrderDate for revenue time-series
```
Reason: OrderDate represents transaction date (GAAP principle)
```

---

## 9. Key Business Insights (Domain Knowledge)

### **9.1 Order Value Drivers**
What makes an order high-value?
1. **More line items** (basket size): Additional products increase SubTotal
2. **Premium products**: Higher UnitPrice products
3. **Quantity**: Bulk purchases (high OrderQty)
4. **Low discounts**: Full-price purchases vs. promotional
5. **Geographic factors**: Territories with higher freight/tax

---

### **9.2 Customer Behavior Patterns**
Observable patterns in real business data:
- **New customers**: Often start with smaller orders (testing)
- **Repeat customers**: Gradually increase order values (trust)
- **Seasonal buyers**: High value during holidays, dormant otherwise
- **Bulk buyers**: Fewer orders but higher values (business customers)
- **Discount seekers**: Only purchase during promotions (lower lifetime value)

---

### **9.3 Product Portfolio Dynamics**
Understanding product contribution:
- **80/20 Rule**: ~20% of products typically drive ~80% of revenue
- **Category concentration**: Some customers buy only from one category
- **Cross-category buyers**: Higher lifetime value (engaged customers)
- **Loss leaders**: Some products sold at low margin to drive traffic
- **Complementary products**: Bikes + accessories purchased together

---

### **9.4 Temporal Patterns**
Time-based business rhythms:
- **Fiscal cycles**: End-of-quarter, end-of-year spikes
- **Seasonal demand**: Spring/summer for bicycles (higher order values)
- **Day-of-week**: Weekday vs. weekend ordering patterns
- **Holiday effects**: Black Friday, Christmas impact
- **Economic cycles**: Recession impact on order values

---

## 10. Reference Quick Guide

### **10.1 Metric Formula Reference**

| Metric | Formula (Conceptual) | Level | Purpose |
|--------|---------------------|-------|---------|
| LineTotal | Qty × Price × (1 - Discount) | Line item | Product revenue contribution |
| SubTotal | Σ LineTotal per order | Order | Pure product revenue |
| TotalDue | SubTotal + Tax + Freight | Order | **Primary order value metric** |
| AOV | Avg(TotalDue) | Segment | Typical order size |
| CLV | Σ TotalDue per customer | Customer | Lifetime revenue |

---

### **10.2 Common Filters**

| Filter Purpose | Condition | Rationale |
|----------------|-----------|-----------|
| Completed orders only | Status = 5 | Only shipped orders count as revenue |
| Exclude cancelled | Status NOT IN (4, 6) | Cancelled/rejected are not sales |
| Current year | YEAR(OrderDate) = 2011 | Time-bound analysis |
| High-value orders | TotalDue > $10,000 | Focus on large transactions |
| Individual customers | CustomerType = 'Individual' | Segment analysis |

---

### **10.3 Relationship Cheat Sheet**

```
Order Value Analysis Navigation:

For "WHO bought?" 
    → SalesOrderHeader.CustomerID → Customer

For "WHAT was bought?"
    → SalesOrderHeader.SalesOrderID → SalesOrderDetail.ProductID → Product

For "WHEN was it bought?"
    → SalesOrderHeader.OrderDate (time dimension)

For "WHERE (geography)?"
    → Customer.TerritoryID → SalesTerritory

For "HOW MUCH?"
    → SalesOrderHeader.TotalDue (complete order value)
    → SalesOrderDetail.LineTotal (per-product contribution)
```

---

## 11. Next Steps for Query Development

### **11.1 Before Writing SQL**
**Checklist:**
- [ ] What is the business question? (Clear objective)
- [ ] What is the grain of analysis? (Order-level, product-level, customer-level)
- [ ] Which metrics are needed? (TotalDue, LineTotal, counts, averages)
- [ ] What filters apply? (Status, date range, customer type)
- [ ] What dimensions for grouping? (Customer, product, time, territory)

### **11.2 Common Query Patterns**
When you're ready to write SQL, typical patterns are:
- **Aggregation**: Group by dimension, aggregate metrics
- **Time-series**: Group by time period, order by date
- **Segmentation**: Filter and compare across segments
- **Ranking**: Top N customers/products by metric
- **Join path**: Header → Detail → Product/Customer for dimensional analysis

---

## 12. Document Revision & Maintenance

**Version:** 1.0  
**Last Updated:** 2025  
**Scope:** AdventureWorks OLTP schema (Sales Order domain)  
**Maintainer:** Research Lab Documentation

**Feedback & Updates:**
- This is a living document
- Report errors or suggest additions based on analytical needs
- Update as new business rules are discovered through query development

---

**End of Documentation**