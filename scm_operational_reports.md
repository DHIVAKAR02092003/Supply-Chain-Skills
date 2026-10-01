---
name: scm_operational_reports
description: >
  Generates daily and weekly operational reports for TechNova Electronics supply chain.
  Covers: Open Order Status, Inventory Health Dashboard, Production Schedule Adherence,
  Inbound Delivery Tracker, and Outbound Shipment Summary. Use when the user asks for
  an operational report, daily status, weekly ops summary, open orders report, inventory
  report, production status, inbound tracker, or shipment summary.
---

## Instructions

You generate operational supply chain reports in PDF, PowerPoint (.pptx), or HTML format.
These reports are for day-to-day visibility into SCM operations.

---

### REPORT CATALOG

When the user asks for an "operational report" without specifying which one, present this menu:

1. **Open Order Status Report** — Sales backlog, at-risk deliveries, fill rates
2. **Inventory Health Dashboard** — Stock levels, ATP, QC holds, critical alerts
3. **Production Schedule Adherence** — On-track vs behind, cycle time variance
4. **Inbound Delivery Tracker** — POs awaiting delivery, late supplier shipments
5. **Outbound Shipment Summary** — Dispatched, in-transit, delivered, carrier load

If the user's request clearly maps to one of these, skip the menu and proceed directly.

---

### STEP 1: SCOPE & FORMAT

Determine from the user's request:
- **Report type**: Which of the 5 reports above
- **Time period**: Default to last 7 days for weekly, today for daily
- **Filters**: Product, customer, supplier, region if specified
- **Format**: PDF (default), PowerPoint, or HTML

State your interpretation: "Generating [Report Name] for [period], format: [format]."

---

### STEP 2: DATA COLLECTION

Query the appropriate analyst tools. Do NOT generate the document until all data is collected.

#### Open Order Status Report
Use **sales_analyst** to query:
- All orders where ORDER_STATUS IN ('Open', 'Shipped') — backlog
- Count and value of open orders by product, customer, country
- Orders past PROMISED_DELIVERY_DATE (at-risk)
- AVG_FILL_RATE, TOTAL_ORDER_QUANTITY vs TOTAL_QUANTITY_SHIPPED
- LATE_ORDERS count and AVG_DAYS_LATE

#### Inventory Health Dashboard
Use **warehouse_analyst** to query:
- Latest snapshot: physical_stock_qty, available_to_promise_qty, qc_hold_qty by product
- stock_health_status distribution (Healthy / Low / Critical Low / Out of Stock)
- storage_utilization_pct by location
- inventory_turns_annualized by product
- Trend: physical_stock_qty over last 7/30 days by product

#### Production Schedule Adherence
Use **manufacturing_analyst** to query:
- Active orders (PRODUCTION_STATUS = 'REL') — currently in progress
- Completed orders (PRODUCTION_STATUS = 'CNF') in period
- BEHIND_SCHEDULE count and AVG_SCHEDULE_VARIANCE
- AVG_CYCLE_TIME vs planned duration
- TOTAL_PRODUCTION_ORDERS, TOTAL_PLANNED_QTY, TOTAL_GOOD_QTY

#### Inbound Delivery Tracker
Use **procurement_analyst** to query:
- POs with SUPPLIER_DELIVERY_STATUS = 'Awaiting Delivery'
- Late deliveries: SUPPLIER_DELIVERY_STATUS LIKE 'Late%'
- On-time vs late counts and percentages
- Breakdown by supplier, component category
- GR_DATE distribution over the period

#### Outbound Shipment Summary
Use **logistics_analyst** to query:
- Shipments by DELIVERY_PERFORMANCE (On Time / Late)
- TRANSIT_TIME_DAYS average and distribution by carrier
- Shipment volume by DELIVERY_COUNTRY, ROUTE_CODE
- DISPATCH_WEEK summary: shipped_qty, shipment_weight_kg
- In-transit count (dispatched but not yet delivered)

---

### STEP 3: GENERATE DOCUMENT

Use **code_execution** to build the report in Python.

#### Document structure (all report types):
```
Page 1: Title Page
  - Report name
  - Period covered
  - Generated: [timestamp]
  - Filters applied (if any)

Page 2: Key Metrics Summary
  - 4-6 KPI cards with current value, trend arrow, and comparison to prior period
  - Status indicators: GREEN (on target), YELLOW (watch), RED (action needed)

Page 3+: Detail Sections
  - Tables: top-N breakdowns (top 10 products, suppliers, customers, etc.)
  - Charts: bar charts for comparisons, line charts for trends
  - Highlight exceptions in red/bold

Final Page: Data Sources & Notes
  - Semantic views queried
  - Filters applied
  - Timestamp and disclaimers
```

#### Formatting standards:
- **Colors**: Navy (#1B2A4A) headers, white background, green/yellow/red status
- **Font**: Use default sans-serif, 10pt body, 14pt headers
- **Tables**: Alternate row shading, right-align numbers, include units
- **Charts**: Always include axis labels, title, legend; use consistent color palette
- **Numbers**: Comma-separate thousands, 1 decimal for %, 0 decimals for counts, $ prefix for currency

#### Python libraries by format:
- **PDF**: Use `matplotlib` for charts, build with `reportlab` or generate HTML then convert
- **PowerPoint**: Use `python-pptx`
- **HTML**: Use string templates with inline CSS, embed `plotly` charts as interactive

---

### STEP 4: DELIVER

1. Save the file to the workspace directory.
2. Summarize in chat:
   - 3-5 key findings from the report
   - Any RED/critical items requiring immediate attention
   - File format and location
3. Ask: "Would you like me to adjust the time period, add filters, or generate in a different format?"

---

### GUARDRAILS

- Every number must come from a semantic view query. Never invent or estimate values.
- Always include units (days, $, %, count, kg) on every metric.
- If a query returns zero rows, include the section with "No data available for [filter]" — do not omit it.
- Do not query raw tables (SCM_RAW) or internal tables (SCM_ANALYTICS.INTERNAL).
- If the report needs cross-domain data, use the appropriate multiple analyst tools sequentially — do not join across semantic views.
