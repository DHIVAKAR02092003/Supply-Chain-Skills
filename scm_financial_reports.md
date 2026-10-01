---
name: scm_financial_reports
description: >
  Generates monthly and quarterly financial and cost reports for TechNova Electronics
  supply chain. Covers: Product Profitability (P&L), Procurement Spend Analysis,
  Inventory Valuation Report, Invoice Reconciliation Report, and Cost-to-Serve Analysis.
  Use when the user asks for a financial report, P&L, profitability report, spend analysis,
  cost report, inventory valuation, invoice reconciliation, cost-to-serve, margin analysis,
  or landed cost breakdown.
---

## Instructions

You generate financial and cost reports for supply chain cost management, profitability
analysis, and accounts payable oversight. These reports support monthly close, quarterly
business reviews, and financial planning.

---

### REPORT CATALOG

When the user asks for a "financial report" or "cost report" without specifying, present this menu:

1. **Product Profitability (P&L)** — Revenue, landed cost breakdown, gross margin by SKU
2. **Procurement Spend Analysis** — Spend by supplier/category/country, price trends
3. **Inventory Valuation Report** — Book value, turns, slow movers, carrying cost
4. **Invoice Reconciliation Report** — 3-way match status, variances, aging
5. **Cost-to-Serve Analysis** — Landed cost per order, freight as % of revenue

---

### STEP 1: SCOPE & FORMAT

Determine:
- **Report type**: Which of the 5 reports
- **Time period**: Default to last calendar month or last quarter
- **Comparison**: Prior period (MoM) or prior year same period (YoY)
- **Filters**: Product, customer, supplier if specified
- **Format**: PDF (default), PowerPoint, or HTML

---

### STEP 2: DATA COLLECTION

#### Product Profitability (P&L)
Use **finance_analyst**:
- Revenue, total_landed_cost, gross_profit, gross_margin_pct by PRODUCT_NAME
- Landed cost breakdown: cost_materials, cost_labor, cost_freight, cost_duty by product
- Revenue_per_unit, landed_cost_per_unit by product
- COST_BASIS distribution (Actual vs Standard) — flag that Actual = MTO (~10%), Standard = SFS (~90%)
- Profitability ranking: products sorted by gross_margin_pct
- Period-over-period: same metrics for prior period
- Customer profitability: gross_margin_pct by CUSTOMER_NAME

#### Procurement Spend Analysis
Use **procurement_analyst**:
- Total spend (sum of line values) by SUPPLIER_NAME, COMPONENT_CATEGORY, SUPPLIER_COUNTRY
- Spend concentration: top 5 suppliers as % of total
- Price trend: average unit price by COMPONENT_CATEGORY over PO_MONTH
- PO volume trend: TOTAL POs by month
- Spend by SUPPLIER_INDUSTRY
- Single-source risk: components with only 1 supplier

Use **finance_analyst** (for landed cost comparison):
- cost_materials vs procurement line value to show PO-to-landed cost uplift

#### Inventory Valuation Report
Use **warehouse_analyst**:
- book_value by PRODUCT_NAME (latest snapshot)
- inventory_turns_annualized by product
- Slow movers: products with inventory_turns_annualized < 4
- stock_health_status distribution by book_value (not just quantity)
- Trend: book_value over time (monthly snapshots)
- storage_utilization_pct

Use **finance_analyst**:
- inventory_book_value, inventory_turns_annualized from finance perspective
- Standard_cost per product

#### Invoice Reconciliation Report
Use **procurement_analyst**:
- THREE_WAY_QTY_MATCH distribution: Match vs Mismatch, by supplier
- THREE_WAY_VALUE_MATCH distribution: Match vs variance amounts, by supplier
- Invoices with mismatches: detail list with PO_NUMBER, SUPPLIER_NAME, variance
- Aging: invoices by INVOICE_DATE grouped into 0-30, 31-60, 61-90, 90+ days
- Match rate trend over PO_MONTH

Use **finance_analyst**:
- INVOICE_MATCH_STATUS distribution (Clean Match / No Invoice / Variance)
- supplier_invoice_amount vs total_landed_cost discrepancies

#### Cost-to-Serve Analysis
Use **finance_analyst** + **logistics_analyst**:

From finance_analyst:
- total_landed_cost, revenue, gross_profit by ORDER_ID
- cost_freight as % of revenue by product and customer
- Landed cost component mix: materials/labor/freight/duty as % of total

From logistics_analyst:
- transit_time_days vs shipment_weight_kg (cost drivers)
- Shipment count and weight by ROUTE_CODE, DELIVERY_COUNTRY

Calculate in code_execution:
- Cost-to-serve per order = total_landed_cost / order_quantity
- Freight intensity = cost_freight / revenue (by product, customer, country)
- Route cost comparison

---

### STEP 3: GENERATE DOCUMENT

Use **code_execution** to build the report.

#### Document structure:
```
Page 1: Title Page
  - Report name, period, generated timestamp
  - Currency: USD ($)

Page 2: Financial Summary
  - Total revenue, total cost, gross profit, gross margin %
  - Period-over-period change with arrows
  - Waterfall chart: Revenue -> Cost components -> Gross Profit

Page 3: Profitability Analysis
  - Ranked table of products/suppliers/customers by margin
  - Stacked bar chart: cost component breakdown by product
  - Scatter plot: revenue vs margin % (bubble size = volume)

Page 4: Trend Analysis
  - Line chart: revenue and margin % over time
  - Line chart: cost component trends
  - Include prior year comparison where available

Page 5: Concentration & Risk
  - Pie/donut chart: spend concentration (top 5 vs rest)
  - Table: single-source dependencies with spend exposure
  - Pareto: top suppliers/products by spend (cumulative %)

Page 6: Exception Detail
  - Table: invoice mismatches, negative-margin products, slow-moving inventory
  - Sorted by financial impact (highest $ first)

Final Page: Assumptions & Data Sources
  - Cost basis methodology (Actual vs Standard)
  - Standard cost table version
  - Semantic views queried
```

#### Financial formatting standards:
- **Currency**: Always show $ with comma-separated thousands. Use 2 decimal places for per-unit, 0 for totals.
- **Percentages**: 1 decimal place (e.g., 34.2%), always show % symbol
- **Negative values**: Red font, parentheses format: ($1,234)
- **Variances**: Show both absolute ($) and relative (%) change
- **Charts**: Waterfall for cost buildup, stacked bar for composition, line for trends
- **Tables**: Right-align all numbers, subtotals in bold, grand total in bold + shaded

---

### STEP 4: DELIVER

1. Save the file to the workspace.
2. Summarize:
   - Top-line P&L: revenue, cost, margin
   - Highest and lowest margin products/customers
   - Key cost drivers or anomalies
   - Action items (invoice mismatches to resolve, slow movers to review)
3. Ask: "Would you like to drill into a specific product, customer, or cost component?"

---

### GUARDRAILS

- Every dollar value must come from a semantic view query. Never estimate.
- Always state the cost basis (Actual vs Standard) when showing cost figures.
- revenue and gross_margin_pct come ONLY from finance_analyst (SV_FINANCE_PERSONA). Do not attempt to calculate revenue from other domains.
- Procurement spend (PO line value) is NOT the same as landed cost. Always label clearly.
- standard_price_per_unit in Warehouse is for valuation only, not PO cost or selling price.
