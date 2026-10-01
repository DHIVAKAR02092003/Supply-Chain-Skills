---
name: scm_performance_reports
description: >
  Generates weekly and monthly performance and KPI reports for TechNova Electronics
  supply chain. Covers: Supplier Scorecard, On-Time Delivery (OTD) Report, Manufacturing
  Yield Report, Carrier Performance Report, and Fill Rate & Backorder Report. Use when
  the user asks for a performance report, KPI report, scorecard, supplier scorecard,
  OTD report, yield report, carrier performance, fill rate report, or backorder analysis.
---

## Instructions

You generate performance and KPI reports that measure supply chain effectiveness against
targets and benchmarks. These reports support weekly reviews and monthly performance meetings.

---

### REPORT CATALOG

When the user asks for a "performance report" or "KPI report" without specifying, present this menu:

1. **Supplier Scorecard** — OTD %, rejection rate, lead time, 3-way match, risk tier
2. **On-Time Delivery (OTD) Report** — Customer OTD %, by region/product/carrier
3. **Manufacturing Yield Report** — First-pass yield, final yield, scrap, rework, defect Pareto
4. **Carrier Performance Report** — Transit time by carrier, OTD by route
5. **Fill Rate & Backorder Report** — Line fill %, order fill %, backorder aging

---

### STEP 1: SCOPE & FORMAT

Determine:
- **Report type**: Which of the 5 reports
- **Time period**: Default to last 30 days (monthly) or last 7 days (weekly)
- **Comparison period**: Prior period for trend comparison (auto-select same length)
- **Filters**: Supplier, product, region, carrier if specified
- **Format**: PDF (default), PowerPoint, or HTML

---

### STEP 2: DATA COLLECTION

#### Supplier Scorecard
Use **procurement_analyst** + **quality_analyst**:

From procurement_analyst:
- On-time delivery rate: count where SUPPLIER_DELIVERY_STATUS = 'On Time' / total delivered, grouped by SUPPLIER_NAME
- Average procurement lead time: days between PO_DATE and GR_DATE, by supplier
- 3-way match rate: count where THREE_WAY_QTY_MATCH = 'Match' AND THREE_WAY_VALUE_MATCH = 'Match' / total, by supplier
- PO volume and spend by supplier
- Same metrics for prior period (for trend arrows)

From quality_analyst:
- Incoming inspection rejection rate by supplier (INSPECTION_TYPE = 'Incoming Inspection')
- Defect code distribution by supplier
- REJECTED_QTY by supplier

Composite scoring (calculate in code_execution):
- **OTD Score** (30% weight): >= 95% = Green, 90-95% = Yellow, < 90% = Red
- **Quality Score** (30% weight): < 2% rejection = Green, 2-5% = Yellow, > 5% = Red
- **Lead Time Score** (20% weight): <= target = Green, 1-2 days over = Yellow, > 2 days over = Red
- **Invoice Match Score** (20% weight): >= 98% = Green, 95-98% = Yellow, < 95% = Red
- **Overall Tier**: Green (all green or 1 yellow), Yellow (2+ yellow or 1 red), Red (2+ red)

#### On-Time Delivery (OTD) Report
Use **sales_analyst** + **logistics_analyst**:

From sales_analyst:
- ON_TIME_STATUS distribution (On Time / Late / In Progress / Backordered)
- OTD rate by CUSTOMER_COUNTRY, PRODUCT_CATEGORY, ORDER_MONTH
- AVG_DAYS_LATE for late orders
- Late order breakdown by customer and product

From logistics_analyst:
- DELIVERY_PERFORMANCE (On Time / Late) by CARRIER_NAME
- DAYS_VS_PROMISE distribution
- OTD by DELIVERY_COUNTRY, ROUTE_CODE

Target benchmark: OTD >= 95%

#### Manufacturing Yield Report
Use **manufacturing_analyst** + **quality_analyst**:

From manufacturing_analyst:
- AVG_FIRST_PASS_YIELD, AVG_FINAL_YIELD by product and production month
- TOTAL_SCRAP_QTY, TOTAL_REWORK_QTY by product
- AVG_MFG_REJECTION_RATE trend
- LOW_YIELD order count
- HAS_SCRAP, HAS_REWORK order counts

From quality_analyst:
- Final Test inspection results (INSPECTION_TYPE = 'Final Test')
- DEFECT_CODE and DEFECT_DETAIL Pareto for manufacturing defects
- REJECTION_RATE_PCT by MATERIAL_NAME for finished products

Target benchmarks: First-pass yield >= 95%, Final yield >= 98%, Scrap rate < 1%

#### Carrier Performance Report
Use **logistics_analyst**:
- AVG transit_time_days by CARRIER_NAME
- OTD (DELIVERY_PERFORMANCE) by carrier
- Shipment volume (shipped_qty, shipment_weight_kg) by carrier
- Transit time by ROUTE_CODE and carrier
- DAYS_VS_PROMISE by carrier (avg, P90)
- Trend: transit time and OTD by week for each carrier

#### Fill Rate & Backorder Report
Use **sales_analyst** + **warehouse_analyst**:

From sales_analyst:
- AVG_FILL_RATE overall and by product
- UNFULFILLED_ORDERS count and value
- LINE_FULFILLMENT distribution (Fulfilled / Partial / None)
- Backorder aging: open orders grouped by days since ORDER_DATE

From warehouse_analyst:
- available_to_promise_qty vs reserved_for_orders_qty by product
- stock_health_status distribution
- Products with stock_health_status IN ('Critical Low', 'Out of Stock')

---

### STEP 3: GENERATE DOCUMENT

Use **code_execution** to build the report.

#### Document structure:
```
Page 1: Title Page
  - Report name, period, generated timestamp

Page 2: Scorecard / KPI Summary
  - KPI tiles with: metric name, current value, target, trend (vs prior period)
  - Color coding: GREEN (meets/exceeds target), YELLOW (within 5% of target), RED (below target)
  - Trend arrows: ▲ improving, ▼ declining, ► stable (< 1% change)

Page 3: Rankings / Comparisons
  - Ranked table (e.g., suppliers ranked by composite score, carriers by OTD)
  - Horizontal bar chart showing performance distribution

Page 4: Trend Analysis
  - Line charts showing weekly/monthly trends for key metrics
  - Include target line as dashed reference

Page 5: Exception Detail
  - Table of items below target with root cause indicators
  - RED items highlighted, sorted worst-first

Page 6: Pareto / Root Cause (where applicable)
  - Pareto chart for defects, late delivery reasons, etc.
  - 80/20 line marked

Final Page: Methodology & Data Sources
  - Scoring methodology (for scorecards)
  - Target benchmarks used
  - Semantic views queried
  - Period and filters
```

#### KPI target benchmarks (use as defaults, state in report):
| Metric | Target | Yellow Zone | Red Zone |
|--------|--------|-------------|----------|
| Customer OTD | >= 95% | 90-95% | < 90% |
| Supplier OTD | >= 95% | 90-95% | < 90% |
| Fill Rate | >= 98% | 95-98% | < 95% |
| First-Pass Yield | >= 95% | 90-95% | < 90% |
| Final Yield | >= 98% | 95-98% | < 95% |
| Rejection Rate | < 2% | 2-5% | > 5% |
| Scrap Rate | < 1% | 1-3% | > 3% |
| Invoice Match | >= 98% | 95-98% | < 95% |

#### Formatting standards:
- **Colors**: Navy (#1B2A4A) headers, green (#27AE60), yellow (#F39C12), red (#E74C3C)
- **Trend arrows**: Green ▲ for improvement, red ▼ for decline, gray ► for stable
- **Tables**: Ranked with position number, alternate row shading, bold header row
- **Charts**: Include target line (dashed black), prior period (light gray), current (navy)

---

### STEP 4: DELIVER

1. Save the file to the workspace.
2. Summarize:
   - Overall performance vs targets (how many KPIs green/yellow/red)
   - Top 3 performers and bottom 3 performers (for ranked reports)
   - Critical items requiring action
3. Ask: "Would you like to drill into any specific supplier/carrier/product, or adjust the targets?"

---

### GUARDRAILS

- Every number must come from a semantic view query. Never invent values.
- Always show both current period AND prior period for comparison.
- Always include the target benchmark and source (state "Target: 95% per SCOR benchmark" etc.).
- If a metric cannot be calculated (missing data), show "N/A" with explanation — do not omit.
- Composite scores must show the breakdown, not just the final tier.
