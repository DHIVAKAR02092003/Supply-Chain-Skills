---
name: scm_executive_reports
description: >
  Generates executive and strategic reports for TechNova Electronics supply chain
  leadership. Covers: Executive SCM Dashboard, Supply Chain Risk Report, Demand vs
  Supply Gap Analysis, Monthly Business Review (MBR), and Customer Service Level Report.
  Use when the user asks for an executive summary, leadership report, MBR, monthly
  business review, risk report, supply chain risk, demand-supply gap, service level
  report, perfect order rate, OTIF report, board report, or C-suite summary.
---

## Instructions

You generate executive-level reports that synthesize cross-domain supply chain data
into strategic summaries for leadership reviews, board presentations, and monthly
business reviews. These reports emphasize insights and recommended actions over raw data.

---

### REPORT CATALOG

When the user asks for an "executive report" or "leadership summary" without specifying, present this menu:

1. **Executive SCM Dashboard** — Top-line KPIs across all 7 domains, 2-3 pages
2. **Supply Chain Risk Report** — Single-source exposure, quality risks, inventory alerts
3. **Demand vs Supply Gap Analysis** — Orders vs ATP vs production capacity
4. **Monthly Business Review (MBR)** — Full period review with trends and action items
5. **Customer Service Level Report** — Perfect order rate, OTIF, customer satisfaction metrics

---

### STEP 1: SCOPE & FORMAT

Determine:
- **Report type**: Which of the 5 reports
- **Time period**: Default to last calendar month; MBR = full month; executive dashboard = last 30 days
- **Audience**: C-suite (high-level), VP/Director (moderate detail), cross-functional team (full detail)
- **Format**: PowerPoint (default for executive), PDF, or HTML

---

### STEP 2: DATA COLLECTION

#### Executive SCM Dashboard
Query ALL analyst tools to collect one headline KPI per domain:

| Domain | Tool | Key Metric |
|--------|------|-----------|
| Sales | sales_analyst | OTD rate, total order value, open orders |
| Procurement | procurement_analyst | Supplier OTD, avg lead time, PO count |
| Manufacturing | manufacturing_analyst | Avg yield, cycle time, production orders completed |
| Logistics | logistics_analyst | Carrier OTD, avg transit time, shipments delivered |
| Warehouse | warehouse_analyst | Avg ATP, stock health distribution, utilization |
| Quality | quality_analyst | Overall rejection rate, QC holds, open notifications |
| Finance | finance_analyst | Total revenue, avg gross margin %, landed cost |

For each: current period value + prior period value + trend direction.

#### Supply Chain Risk Report
Use **procurement_analyst** + **quality_analyst** + **warehouse_analyst**:

From procurement_analyst:
- Single-source risk: components supplied by only 1 supplier
- Supplier concentration: top 3 suppliers as % of total spend
- Chronically late suppliers: OTD < 90% over 90 days

From quality_analyst:
- RED-tier suppliers: rejection_rate_pct > 5%
- Repeat defects: same defect code 3+ times in 90 days
- QC holds impacting saleable inventory

From warehouse_analyst:
- stock_health_status = 'Critical Low' or 'Out of Stock' products
- Products with declining ATP trend (last 30 days)

Use **Documents_search**:
- Search: "Third-Party Risk Management Standard" for risk classification criteria
- Search: "Responsible Sourcing & Supplier Code" for compliance risks

Risk matrix (calculate in code_execution):
- **Likelihood** (1-5): based on frequency of issues
- **Impact** (1-5): based on spend exposure and production dependency
- **Risk Score**: Likelihood x Impact
- Categorize: Critical (20-25), High (12-19), Medium (6-11), Low (1-5)

#### Demand vs Supply Gap Analysis
Use **sales_analyst** + **warehouse_analyst** + **manufacturing_analyst**:

From sales_analyst:
- TOTAL_ORDER_QUANTITY by PRODUCT_NAME (demand signal)
- OPEN_ORDERS quantity (unfulfilled demand)
- Order trend by ORDER_MONTH

From warehouse_analyst:
- available_to_promise_qty by product (current supply)
- physical_stock_qty trend

From manufacturing_analyst:
- TOTAL_PLANNED_QTY, TOTAL_GOOD_QTY by product (production output)
- PRODUCTION_STATUS = 'REL' orders (pipeline)

Gap calculation (in code_execution):
- Demand = open order qty + projected orders (based on trend)
- Supply = current ATP + in-production qty
- Gap = Demand - Supply (negative = shortfall)
- Days of supply = ATP / (avg daily demand)

#### Monthly Business Review (MBR)
Query ALL analyst tools — this is the most comprehensive report.

Collect for the full calendar month:
1. **Sales**: total orders, revenue, OTD, fill rate, new customers
2. **Procurement**: POs raised, spend, supplier OTD, avg lead time
3. **Manufacturing**: orders completed, yield, cycle time, scrap/rework
4. **Logistics**: shipments, transit time, carrier OTD, delivery by region
5. **Warehouse**: avg stock levels, turns, utilization, health distribution
6. **Quality**: inspections, rejection rates (incoming + final), top defects
7. **Finance**: revenue, gross profit, margin %, cost breakdown

For each domain: current month vs prior month vs same month prior quarter.

Use **Documents_search** for any compliance events or policy references relevant to the period.

#### Customer Service Level Report
Use **sales_analyst** + **logistics_analyst** + **warehouse_analyst**:

From sales_analyst:
- ON_TIME_STATUS distribution
- AVG_FILL_RATE by customer and product
- LATE_ORDERS, AVG_DAYS_LATE

From logistics_analyst:
- DELIVERY_PERFORMANCE by customer
- DAYS_VS_PROMISE distribution
- Transit time by destination country

Perfect Order calculation (in code_execution):
- **On-Time**: delivered by PROMISED_DELIVERY_DATE
- **In-Full**: LINE_FULFILLMENT = 'Fulfilled'
- **No Quality Issue**: no QC hold on delivered product (from warehouse_analyst)
- **Perfect Order Rate** = (On-Time AND In-Full AND No Defect) / Total Orders
- OTIF (On-Time In-Full) = On-Time AND In-Full / Total Orders

---

### STEP 3: GENERATE DOCUMENT

Use **code_execution** to build the report.

#### Document structure (Executive / MBR):
```
Slide/Page 1: Title
  - Report name, period, "TechNova Electronics Inc."
  - Audience level noted

Slide/Page 2: Executive Summary
  - 3-5 bullet points: key wins, key risks, recommended actions
  - Written in executive language: concise, action-oriented
  - Overall health indicator: GREEN / YELLOW / RED with 1-line rationale

Slide/Page 3: KPI Scorecard
  - Grid layout: 7 domain tiles, each showing:
    - Domain name + icon/color
    - Primary KPI value
    - Trend arrow + % change vs prior period
    - Traffic light: GREEN/YELLOW/RED
  - For MBR: also show target vs actual

Slide/Page 4: Performance Trends
  - Multi-line chart: 3-4 most critical KPIs over 6-12 months
  - Annotate inflection points or anomalies
  - Include target reference lines

Slide/Page 5: Risk Matrix (for risk report)
  - 5x5 heatmap: Likelihood vs Impact
  - Plotted risks with labels
  - Top 5 risks listed with mitigation recommendations

Slide/Page 6: Demand-Supply View (for gap analysis)
  - Stacked area chart: demand vs supply over time
  - Table: gap by product with days-of-supply
  - RED highlight for products in shortfall

Slide/Page 7-8: Domain Deep Dives (MBR only)
  - One section per domain with 3-5 KPIs, small chart, and commentary
  - Exception callouts in red boxes

Slide/Page 9: Actions & Recommendations
  - Numbered action items with owner suggestion (domain)
  - Priority: Critical / High / Medium
  - "Decided" vs "For Discussion" categorization

Final Slide/Page: Appendix
  - Data sources (semantic views)
  - Methodology notes
  - Glossary of terms/acronyms
```

#### Executive formatting standards:
- **Layout**: Clean, minimal, maximum white space. No more than 6 data points per page.
- **Colors**: Navy (#1B2A4A) primary, white background, accent green/yellow/red for status
- **Font**: 24pt slide titles, 18pt section headers, 14pt body (PowerPoint). Proportionally smaller for PDF.
- **Charts**: Simple, labeled, no 3D effects. One chart per concept.
- **Text**: Bullet points max 2 lines each. No paragraphs on slides.
- **Numbers**: Round to whole numbers for executives (not $1,234,567.89 — use $1.2M)
- **Abbreviations**: Define on first use; include glossary in appendix

---

### STEP 4: DELIVER

1. Save the file to the workspace.
2. Summarize in executive style:
   - Overall supply chain health (1 sentence + GREEN/YELLOW/RED)
   - Top 3 wins this period
   - Top 3 risks or action items
   - One recommended strategic decision
3. Ask: "Would you like me to adjust the detail level, add a specific domain deep-dive, or generate talking points for the presentation?"

---

### GUARDRAILS

- Executive reports must be CONCISE. 2-3 pages for dashboards, 5-8 pages for MBR, max 10 for full MBR with appendix.
- Every number must trace to a semantic view query. State "Source: SV_[DOMAIN]_PERSONA" in appendix.
- Round large numbers for readability: use K, M, B suffixes ($1.2M, 3.4K units).
- Do NOT present raw query results. Always synthesize into insights and recommendations.
- Cross-domain data: query each domain tool separately. Never join across semantic views.
- Recommendations must be grounded in the data presented. Do not suggest actions unsupported by the metrics.
- Perfect Order Rate calculation must clearly state the components and how each was measured.
- MBR commentary should be neutral and factual, not optimistic or pessimistic. State what changed and by how much.
