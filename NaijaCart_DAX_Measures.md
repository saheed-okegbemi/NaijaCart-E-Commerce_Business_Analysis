# 📐 NaijaCart DAX Measures Plan 

Trimmed to exactly what the locked 3-page dashboard layout needs — every measure below is tied to a specific KPI card or chart in the Dashboard Build Guide. **15 measures + 1 calculated column, total.**

**Relationships:**
`Orders` (1) → `OrderLines` (Many) · `Orders` (1) → `Delivery` (One-to-one, non-cancelled only) · `Date` (1) → `Orders[OrderDate]` (Many) · `Customers`, `Stores`, `Employees`, `Promotions`, `PaymentMethods` all (1) → `Orders` (Many) · `Products` (1) → `OrderLines` (Many) · `Riders` (1) → `Delivery` (Many)

---

## 📊 Dashboard 1: Sales and Revenue Performance (6 measures)

### KPI Cards

```dax
Total Revenue =
CALCULATE ( SUM ( OrderLines[LineTotal] ), Orders[OrderStatus] <> "Cancelled" )
```

```dax
Total Orders =
CALCULATE ( DISTINCTCOUNT ( Orders[OrderID] ), Orders[OrderStatus] <> "Cancelled" )
```

```dax
Average Order Value =
DIVIDE ( [Total Revenue], [Total Orders] )
```

```dax
Total Cost =
CALCULATE (
    SUMX ( OrderLines, OrderLines[Quantity] * RELATED ( Products[CostPrice] ) ),
    Orders[OrderStatus] <> "Cancelled"
)
```

```dax
Gross Profit =
[Total Revenue] - [Total Cost]
```

**KPI row (5 cards):** Total Revenue · Total Orders · Average Order Value · Gross Profit · Total Cost

> Profit uses `LineTotal` (which already reflects the discount applied to revenue) minus `Quantity × CostPrice` (which is unaffected by discount) — so a heavily discounted line correctly shows a thinner margin, not just lower revenue.

### Ranking (for Top 10 Stores chart)

```dax
Store Revenue Rank =
RANKX ( ALL ( Stores[StoreName] ), [Total Revenue], , DESC )
```

*Used as an "Advanced filtering" visual-level filter (`Store Revenue Rank <= 10`) on the Top 10 Stores chart, instead of Power BI's built-in Top N filter type. Built-in Top N filters don't respond to cross-filtering from clicking another visual (e.g. selecting a state on the map) — only to slicers and page/report filters. `ALL(Stores[StoreName])` only clears the filter on store name, so a state selection on the map still narrows the ranking pool correctly, while `[Total Revenue]` itself still only includes non-cancelled orders.*

### Charts 
- **Revenue trend over time** → line chart plotting `[Total Revenue]` and `[Gross Profit]` together by `Date` hierarchy, **expanded** (not drilled) to Year + MonthName — see Build Guide Note 6
- **Revenue by region/state** → slice `[Total Revenue]` by `Stores[State]`
- **Top 10 stores by revenue** → slice `[Total Revenue]` by `Stores[StoreName]`, filtered using `[Store Revenue Rank] <= 10` (see below) rather than a built-in Top N filter
- **Order status breakdown** → plain `COUNT` of `Orders[OrderID]` by `Orders[OrderStatus]`
- **Profit by region** → clustered column chart, slice `[Gross Profit]` by `Stores[Region]`

---

## 🛒 Dashboard 2: Product and Customer Analysis (3 measures + 1 calculated column)

```dax
Total Quantity Sold =
CALCULATE ( SUM ( OrderLines[Quantity] ), Orders[OrderStatus] <> "Cancelled" )
```

```dax
Total Customers =
CALCULATE ( DISTINCTCOUNT ( Orders[CustomerID] ), Orders[OrderStatus] <> "Cancelled" )
```

```dax
Avg Revenue per Customer =
DIVIDE ( [Total Revenue], [Total Customers] )
```

**KPI row (4 cards):** Total Revenue · Total Quantity Sold · Total Customers · Avg Revenue per Customer

### Calculated Column (for Discount Impact chart)

```dax
DiscountStatus =
IF ( OrderLines[DiscountRate] > 0, "Discounted", "Full-Price" )
```

*Added directly on the `OrderLines` table (Modeling → New Column). This is a calculated column, not a measure — it needs to exist as a physical value per row so it can sit on a chart axis.*

### Charts

- **Revenue by category/subcategory** → slice `[Total Revenue]` by `Products[Category]`
- **Top 10 products by revenue** → slice `[Total Revenue]` by `Products[ProductName]`, Top N filter
- **Top 10 customers by spend** → slice `[Total Revenue]` by `Customers[CustomerName]`, Top N filter
- **Revenue by customer segment** → slice `[Total Revenue]` by `Customers[CustomerSegment]`
- **Discount impact** → line and clustered column combo chart, `[Total Revenue]` on columns (primary axis), `[Total Quantity Sold]` on line (secondary axis), by `OrderLines[DiscountStatus]`

---

## 🚚 Dashboard 3: Delivery and Logistics Performance (6 measures)

```dax
Total Deliveries =
COUNTROWS ( Delivery )
```

```dax
Delivered Count =
CALCULATE ( COUNTROWS ( Delivery ), Delivery[DeliveryStatus] = "Delivered" )
```

```dax
Failed Count =
CALCULATE ( COUNTROWS ( Delivery ), Delivery[DeliveryStatus] = "Failed" )
```

```dax
Delivery Success Rate =
DIVIDE ( [Delivered Count], [Total Deliveries] )
```

```dax
Delivery Failure Rate =
DIVIDE ( [Failed Count], [Total Deliveries] )
```

```dax
Average Delivery Time (Hrs) =
CALCULATE ( AVERAGE ( Delivery[DeliveryTimeHours] ), Delivery[DeliveryStatus] = "Delivered" )
```

**KPI row (4 cards):** Total Deliveries · Delivery Success Rate · Average Delivery Time (Hrs) · Delivery Failure Rate

### Charts 
- **Delivery status breakdown** → plain `COUNT` of `Delivery[DeliveryID]` by `Delivery[DeliveryStatus]`
- **Avg delivery time by state** → slice `[Average Delivery Time (Hrs)]` by `Delivery[DestinationState]`
- **Rider/logistics partner ranking** → `[Delivery Success Rate]` sliced by `Riders[RiderName]` / `Riders[LogisticsPartner]`
- **Delivery outcome by vehicle type** → plain `COUNT` of `Delivery[DeliveryID]` by `Riders[VehicleType]`, legend `Delivery[DeliveryStatus]`

---

## ⚠️ Modeling Notes

1. **Cancelled orders are excluded from revenue, cost, and profit** — all filter out `OrderStatus = "Cancelled"`.
2. **`OrderLines` doesn't carry `OrderStatus`** directly — measures rely on the `Orders → OrderLines` relationship (single-direction, Orders filters OrderLines) propagating the status filter down. Confirm this direction in Model view.
3. **`Total Cost` requires `OrderLines → Products` to be an active many-to-one relationship** for `RELATED()` to resolve `CostPrice`.
4. **`Delivery` excludes cancelled orders by design** (13,000 of 15,500 orders) — `Total Deliveries` will always be lower than `Total Orders`, expected.
5. Donut charts (Order Status, Delivery Status) use plain `COUNT`, not the filtered measures — this is intentional so cancelled/failed records still appear in those breakdowns.
6. **`Store Revenue Rank` deliberately avoids Power BI's built-in Top N visual filter** — Top N filters don't recalculate in response to cross-filtering from another visual's selection (e.g. clicking a state on the map), only from slicers and page/report-level filters. The rank-measure-plus-Advanced-filter approach works correctly in both cases.
