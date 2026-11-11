# Sales and Budget Analysis (Power BI)

A one-page Power BI dashboard comparing actual **sales** against the planned **budget** for the AdventureWorks bike retailer. It shows where the business is ahead of or behind plan, by product category, by month, and which products and customers drive revenue.

Static export: [Sales and Budget Analysis.pdf](Sales%20and%20Budget%20Analysis.pdf)

## Headline results

| Metric    | Value          |
|-----------|----------------|
| Sales     | $16,473,618    |
| Budget    | $16,869,574    |
| Variance  | ($395.96K)     |
| Variance% | -2.3%          |

Sales finished about **$396K (2.3%) below budget**.

- **Bikes** account for almost all revenue. Accessories and Clothing are small by comparison.
- **Mountain-200** models dominate the top-products list: the six best sellers are all Mountain-200 variants at roughly $0.9–1.0M each, followed by Touring-1000 and Road-350-W.
- Monthly variance stays **negative throughout the year**, so the gap is a steady miss rather than one bad month.
- Revenue is spread across many customers. The top customer (Jordan Turner) spent $11,484, and the top 10 are all between $9.6K and $11.5K.

## Repository contents

| File | Description |
|------|-------------|
| `Sales and Budget Analysis.pbix` | Power BI Desktop report (data model, Power Query, visuals) |
| `Sales and Budget Analysis.pdf`  | Static PDF export of the dashboard |
| `AdventureWorks_Database.xlsx`   | Source data: sales transactions and lookup tables |
| `Budget.xlsx`                    | Source data: monthly budget per product for 2016 |

## Data model

Six tables in a star schema, with `Sales` and `Budget` as fact tables sharing the `Product` and `Calendar` dimensions:

| Table | Source | Rows | Key columns |
|-------|--------|-----:|-------------|
| Sales       | AdventureWorks_Database.xlsx | 58,189 | ProductKey, CustomerKey, SalesTerritoryKey, OrderDate, OrderQuantity, UnitPrice, SalesAmount |
| Budget      | Budget.xlsx                  | product × month | ProductKey, Month, BudgetAmount |
| Product     | AdventureWorks_Database.xlsx | 606    | ProductKey, ProductName, Category, SubCategory, Color, ListPrice |
| Customer    | AdventureWorks_Database.xlsx | 18,484 | CustomerKey, FullName, Gender, YearlyIncome, Occupation |
| Calendar    | AdventureWorks_Database.xlsx | 1,461  | Date, Year, Quarter, Month, FiscalYear |
| Territories | AdventureWorks_Database.xlsx | 11     | SalesTerritoryKey, Region, Country, Group |

```
Customer ─┐                ┌─ Calendar
          ├── Sales ───────┤
Territories┘      │        │
               Product ── Budget
```

### Power Query transformations

The **Budget** sheet comes in as a wide pivot table (one column per month, Jan–Dec 2016) and is reshaped into a tidy fact table:

1. Skip the 3 header rows above the table and promote the real headers.
2. Filter out subtotal rows (any `Category` containing "Total").
3. Drop the `Grand Total` column.
4. **Unpivot** the 12 month columns into `Month` / `BudgetAmount` rows.
5. Convert `Month` to a date so it can join to `Calendar`.

The AdventureWorks tables are loaded from named Excel tables, with only data-type changes applied.

### Measures

| Measure     | Meaning |
|-------------|---------|
| `Sales`     | Total of `Sales[SalesAmount]` |
| `Budget`    | Total of `Budget[BudgetAmount]` |
| `Variance`  | `Sales - Budget` |
| `Variance%` | `Variance / Budget` |

## Dashboard layout

- **KPI cards:** Sales, Budget, Variance, Variance%
- **Sales and Budget by Category:** clustered bar chart, drillable to SubCategory
- **Variance by Category:** bar chart, drillable to SubCategory
- **Sales and Budget by Month:** line chart
- **Variance by Month:** column chart
- **Top Products:** table of ProductName and Sales
- **Top Customers:** table of FullName and Sales

## How to open

1. Install [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (Windows).
2. Open `Sales and Budget Analysis.pbix`. The data is already loaded in the file, so it can be viewed without refreshing.
3. To **refresh** the data, point the queries at your local copies of the Excel files. The queries still use the original author's paths (`C:\Users\jerom\Downloads\Downloads\...`). In Power BI, go to **Transform data → Data source settings → Change Source** and select `AdventureWorks_Database.xlsx` and `Budget.xlsx` from this folder.

No Power BI? Open `Sales and Budget Analysis.pdf` to see the finished dashboard.

## Tools

Power BI Desktop · Power Query (M) · DAX · Excel
