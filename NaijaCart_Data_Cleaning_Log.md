# 🧹 NaijaCart Data Cleaning Log

This log documents the data quality assessment performed on the raw NaijaCart tables and the transformation steps applied in **Power Query** before loading the data into the Power BI model. All cleaning was done at the source-query level (transform-before-load) so the model only ever holds clean, analysis-ready tables.

---

## 1. Data Quality Assessment

Before building any visuals, each table was profiled in Power Query (Column Quality, Column Distribution, Column Profile) to check for missing values, duplicates, and formatting inconsistencies.

### Findings

| Table | Column | Issue | Rows Affected |
|---|---|---|---|
| `Orders` | `EmployeeID` | Missing (blank) values | 51 of 15,500 (0.33%) |
| `Orders` | `PaymentMethodID` | Missing (blank) values | 52 of 15,500 (0.34%) |
| `Orders` | `StoreID` | Leading/trailing whitespace | 78 of 15,500 (0.50%) |
| `OrderLines` | (all columns) | Fully duplicated rows (excluding the line ID) | 80 of 26,239 (0.30%) |

No missing values were found in critical numeric fields (`OrderTotal`, `LineTotal`, `UnitPrice`, `Quantity`), and no orphaned foreign keys were found (every non-blank `CustomerID`, `StoreID`, `ProductID`, `PromotionID` in the fact tables has a matching row in its dimension table).

---

## 2. Cleaning Strategy

Different treatment was applied depending on the role of each field, consistent with standard practice for transactional retail data:

- **Non-critical categorical keys with missing values** (`EmployeeID`, `PaymentMethodID`) → replaced with `"UNKNOWN"` rather than dropped, so the underlying order (a valid transaction) is preserved. This keeps `Orders` at its full, correct row count while making incomplete records identifiable in any employee- or payment-level breakdown.
- **Whitespace inconsistencies** (`StoreID`) → trimmed. Left uncleaned, these create phantom duplicate store groups (e.g. `"STO-008"` and `" STO-008 "` treated as different stores in a relationship or `GROUP BY`).
- **Duplicate transactional rows** (`OrderLines`) → removed. A duplicated line item would double-count quantity and revenue in any measure built on `OrderLines`.

---

## 3. Power Query Transformation Steps

Applied in the **Power Query Editor**, per table, before loading to the model.

### `Orders` query

| Step | Transformation | Why |
|---|---|---|
| 1 | `Trim` on `StoreID` (Transform → Format → Trim) | Removes leading/trailing whitespace so every `StoreID` matches its `Stores` dimension key exactly. |
| 2 | Replace Values: blank/null → `"UNKNOWN"` on `EmployeeID` | Preserves the order record while flagging that no employee was captured. |
| 3 | Replace Values: blank/null → `"UNKNOWN"` on `PaymentMethodID` | Same rationale — keeps the transaction, flags the gap. |
| 4 | Change Type: confirm `OrderDate` is `Date`, `ShippingFee`/`OrderTotal` are `Decimal Number` | Ensures correct sort order and aggregation behavior downstream. |

**M code reference:**
```m
= Table.TransformColumns(Source, {{"StoreID", Text.Trim, type text}})
```
```m
= Table.ReplaceValue(Source, null, "UNKNOWN", Replacer.ReplaceValue, {"EmployeeID"})
```
```m
= Table.ReplaceValue(Source, null, "UNKNOWN", Replacer.ReplaceValue, {"PaymentMethodID"})
```

### `OrderLines` query

| Step | Transformation | Why |
|---|---|---|
| 1 | Remove Duplicates (Home → Reduce Rows → Remove Rows → Remove Duplicates), applied on all columns **except** `OrderLineID` | `OrderLineID` is a synthetic row identifier, so comparing including it would never catch a duplicate. Excluding it correctly flags true duplicate transactions. |
| 2 | Change Type: `Quantity` as `Whole Number`; `UnitPrice`, `DiscountRate`, `LineTotal` as `Decimal Number` | Ensures correct aggregation and avoids implicit text-to-number conversion errors. |

**Approach for duplicate removal (dedupe on a subset of columns):**
```m
= Table.Distinct(Source, {"OrderID", "ProductID", "Quantity", "UnitPrice", "DiscountRate", "LineTotal"})
```

### `Employees`, `Customers`, `Stores`, `Products`, `Promotions`, `PaymentMethods`, `Riders`, `Delivery`, `Date`

| Step | Transformation | Why |
|---|---|---|
| 1 | Confirm data types per column (Text, Date, Decimal Number, Whole Number, per the Data Dictionary) | Power Query sometimes infers types incorrectly from source files (e.g. reading an ID column as a number). Explicit typing avoids silent join failures in the model. |
| 2 | Trim + Clean on all text key columns (`Text.Trim`, `Text.Clean`) | Defensive step — catches any stray whitespace or non-printable characters not surfaced during profiling. |

No missing values, duplicates, or blank keys were found in these tables during profiling, so no corrective steps beyond type-checking and defensive trimming were required.

### `Stores`, `Customers`, `Delivery` — Shape Map compatibility

| Step | Transformation | Why |
|---|---|---|
| 1 | Replace Values: `"FCT"` → `"Federal Capital Territory"` on `Stores[State]`, `Customers[State]`, and `Delivery[DestinationState]` | The dataset stores Abuja's state as the common abbreviation `"FCT"`, but third-party Nigeria map files (e.g. geoBoundaries) label this region by its full name, `"Federal Capital Territory"`, in their location property. Without this fix, any Shape Map / Filled Map visual will fail to match this state and show it as blank or "undefined" on hover. |

**M code reference (repeat per table/column):**
```m
= Table.ReplaceValue(Source, "FCT", "Federal Capital Territory", Replacer.ReplaceValue, {"State"})
```
*(for `Delivery`, use `{"DestinationState"}` as the column list instead of `{"State"}`)*

> ⚠️ Confirm the exact spelling used by whichever map file you download — some sources use `"Federal Capital Territory"`, others `"FCT Abuja"` or `"Abuja Federal Capital Territory"`. Open the map file in a text editor and search for "Abuja" to check the exact property value before finalizing this replace step.

---

## 4. Post-Cleaning Validation

After applying the above steps, the following checks confirm the model is ready to load:

- `Orders` row count remains **15,500** (no rows dropped — only values replaced).
- `OrderLines` row count reduces from **26,239 → 26,159** (80 duplicate rows removed).
- `Orders[StoreID]` values match `Stores[StoreID]` exactly on every row (no orphaned keys after trimming).
- No blank values remain in `Orders[EmployeeID]` or `Orders[PaymentMethodID]` (`"UNKNOWN"` used consistently, matching the convention already used by `Promotions[NO_PROMO]`).
- Referential date logic holds across all tables (see the Data Dictionary's *Date Integrity Checks* section): no order precedes its customer's signup date or its employee's hire date, no delivery precedes its dispatch date, and all delivery dates fall within the `Date` dimension's range.
- `"FCT"` no longer appears as a value in `Stores[State]`, `Customers[State]`, or `Delivery[DestinationState]` — all replaced with `"Federal Capital Territory"` (or the exact match required by your chosen map file) so map visuals render every state correctly.

---

## 5. Load

Once each query passes validation, click **Close & Apply** to load the cleaned tables into the Power BI model, then proceed to building relationships per the Data Dictionary's relationship map.
