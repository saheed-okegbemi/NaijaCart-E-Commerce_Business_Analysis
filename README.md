# 🛒 NaijaCart E-Commerce Performance Analysis

## 📊 Project Overview

NaijaCart is a Power BI project built around a fictional Nigerian e-commerce business. Instead of working from a single flat spreadsheet, I designed a relational dataset with 15,500 orders spread across 11 connected tables covering customers, products, stores, promotions, payments, employees, riders, and delivery outcomes.

The goal was to go past just building charts and actually think through the data the way a retail analyst would: model the relationships properly in Power BI, write DAX that holds up under cross-filtering, and build a three-page dashboard that answers real questions about sales, customers, and logistics rather than just displaying numbers.

---

## 🎯 Business Questions

**Sales and Revenue**
- How much revenue, profit, and orders has NaijaCart generated, and how has that changed over time?
- Which states and stores are driving the most revenue, and which are lagging?
- How much of total revenue comes from promotional campaigns versus regular pricing?
- What share of orders end up completed, cancelled, or returned?

**Product and Customer**
- Which product categories and individual products sell the most?
- Who are the highest-spending customers, and how does revenue break down across customer segments?
- Does discounting actually move more product, or does full-price product still carry the business?

**Delivery and Logistics**
- What percentage of deliveries succeed, get delayed, or fail outright?
- Which states see the longest delivery times?
- How do riders and logistics partners compare on delivery success, and does vehicle type matter?

---

## 🛠️ Tools Used

- **Power BI**: data modeling, DAX, dashboard design
- **Power Query**: data cleaning and transformation before load
- **DAX**: measures for revenue, profit, delivery performance, and ranking
- **xlsx**: source data, structured as a relational star schema across 11 tables

---

## 📈 Dashboard Pages

### 1. Sales and Revenue Performance

KPI cards for total revenue, orders, average order value, gross profit, and total cost, alongside a revenue and profit trend line, a state-level revenue map, top-performing stores, profit by region, and an order status breakdown.

### 2. Product and Customer Analysis

Covers revenue by category and subcategory, top products, top customers, revenue by customer segment, and a discount impact chart comparing revenue and quantity between discounted and full-price orders.

### 3. Delivery and Logistics Performance

Tracks delivery outcomes, average delivery time by state, a rider and logistics partner performance ranking, and delivery success broken down by vehicle type.

---

## 💡 Key Findings

- Total revenue came to ₦4,175,844,875.80 across 13,000 completed and returned orders, with an average order value of roughly ₦321,000
- Gross profit landed at ₦1.02 billion against ₦3.16 billion in cost
- Lagos was the strongest state by a clear margin, while Kebbi came in weakest
- South West led both revenue and profit at the regional level, and North Central trailed, which isn't surprising given it's represented by only one state, the FCT
- The gap between the top and tenth-best store was narrow, around ₦159 million versus ₦148 million, so performance at the top of the store list is fairly even rather than dominated by one outlier
- Cancellations and returns combined came to roughly a third of all orders, higher than expected and worth digging into further in a real business setting
- Computing was the strongest product category by revenue at ₦1.4 billion, with Electronics and Phones & Tablets close behind
- Groceries brought up the rear at just ₦18 million, which tracks given how cheap individual grocery items are compared to laptops and phones
- The single best-selling product was the Asus Laptop Pro
- New customers accounted for the largest share of revenue at 47%, followed by Returning at 40% and VIP at 13%
- Full-price orders outsold discounted ones on both revenue and quantity
- Delivery performance held up reasonably well: an 89% success rate, a 2.5% failure rate, and average delivery times around 61 hours
- Motorbike deliveries performed best overall, while bicycle deliveries lagged behind the other vehicle types, which raises a fair sustainability question about how much the business should lean on lower-emission options going forward

---

## 🔗 Links

- **Medium article**: [Building NaijaCart: A Power BI Analysis of a Nigerian E-Commerce Business](https://medium.com/@yemtech96/building-naijacart-a-power-bi-analysis-of-a-nigerian-e-commerce-business-020e2b88e1dc)
- **Interactive dashboard**: [View on Power BI](https://app.powerbi.com/reportEmbed?reportId=264febf2-75fc-4f2f-8ebe-4e56fce690f0&autoAuth=true&ctid=fccf42df-16d9-4e87-a6e0-e49622017f74)
- **Data dictionary**: see `NaijaCart_Data_Dictionary.md` in this repo for the full field-by-field breakdown of all 11 tables
