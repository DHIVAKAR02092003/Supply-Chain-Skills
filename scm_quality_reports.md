---
name: scm_quality_reports
description: >
  Generates quality and compliance reports for TechNova Electronics supply chain.
  Covers: Incoming Quality Inspection Summary, Final Test Quality Report, Supplier
  Audit Readiness Report, and CAPA Tracker. Use when the user asks for a quality report,
  inspection report, defect analysis, rejection rate report, supplier audit report,
  CAPA report, corrective action report, compliance report, or quality trend analysis.
---

## Instructions

You generate quality and compliance reports covering incoming material inspections,
final product testing, supplier quality performance, and corrective action tracking.
These reports support quality review meetings, supplier audits, and compliance documentation.

---

### REPORT CATALOG

When the user asks for a "quality report" without specifying, present this menu:

1. **Incoming Quality Inspection Summary** — Supplier rejection rates, defect Pareto, trends
2. **Final Test Quality Report** — Assembly defect rates, product yield, failure analysis
3. **Supplier Audit Readiness Report** — Supplier risk profile with policy citations
4. **CAPA Tracker** — Quality notifications, repeat defects, resolution status

---

### STEP 1: SCOPE & FORMAT

Determine:
- **Report type**: Which of the 4 reports
- **Time period**: Default to last 30 days; for audit readiness, last 90 days
- **Filters**: Supplier, material/product, defect code, plant if specified
- **Format**: PDF (default), PowerPoint, or HTML

---

### STEP 2: DATA COLLECTION

#### Incoming Quality Inspection Summary
Use **quality_analyst**:
- Filter: INSPECTION_TYPE = 'Incoming Inspection'
- REJECTION_RATE_PCT by SUPPLIER_NAME — rank worst to best
- REJECTED_QTY and total inspected by MATERIAL_CATEGORY
- DEFECT_CODE and DEFECT_DETAIL frequency — Pareto distribution
- Trend: REJECTION_RATE_PCT by INSPECTION_MONTH and supplier
- Current QC holds: CURRENT_QC_HOLD_QTY by material
- Risk tier classification: GREEN (< 2%), YELLOW (2-5%), RED (> 5%)

Use **procurement_analyst** (for supplier context):
- SUPPLIER_COUNTRY, SUPPLIER_INDUSTRY for each flagged supplier
- PO volume per supplier (to weight rejection impact)

#### Final Test Quality Report
Use **quality_analyst**:
- Filter: INSPECTION_TYPE = 'Final Test'
- REJECTION_RATE_PCT by MATERIAL_NAME (finished products)
- DEFECT_CODE and DEFECT_DETAIL Pareto for assembly defects
- REJECTED_QTY by PRODUCTION_ORDER_NUMBER
- Trend: REJECTION_RATE_PCT by INSPECTION_MONTH

Use **manufacturing_analyst**:
- AVG_FIRST_PASS_YIELD, AVG_FINAL_YIELD by PRODUCT_NAME
- TOTAL_SCRAP_QTY, TOTAL_REWORK_QTY, TOTAL_TEST_FAILURES
- LOW_YIELD production orders — detail list
- AVG_MFG_REJECTION_RATE trend by PRODUCTION_MONTH

#### Supplier Audit Readiness Report
Use **quality_analyst**:
- 90-day rejection rate trend by supplier
- All defect codes and frequencies per supplier
- QC hold history per supplier's materials

Use **procurement_analyst**:
- Supplier OTD rate (SUPPLIER_DELIVERY_STATUS) over 90 days
- 3-way match rate per supplier
- PO volume and spend (scale of relationship)

Use **Documents_search** (for policy citations):
- Search for: "Supplier Audit & Compliance Assessment Procedure" (Doc 07)
- Search for: "Third-Party Risk Management Standard" (Doc 03)
- Search for: "Supplier Security Engineering Requirements" (Doc 01)
- Extract relevant audit criteria, thresholds, and required documentation

Composite audit readiness profile (calculate in code_execution):
- **Quality Score**: rejection rate vs threshold (GREEN/YELLOW/RED)
- **Delivery Score**: OTD % vs 95% target
- **Financial Score**: invoice match rate vs 98% target
- **Compliance Checklist**: per Doc 07 criteria (present as checklist items)
- **Overall Readiness**: Ready / Needs Attention / High Risk

#### CAPA Tracker (Corrective & Preventive Action)
Use **quality_analyst**:
- All quality notifications in period
- Group by NOTIFICATION_STATUS (COMP = completed)
- Repeat defects: same DEFECT_CODE appearing for same SUPPLIER_NAME or MATERIAL_NAME 2+ times
- Time to resolution: INSPECTION_DATE to notification closure (if available)
- Severity: REJECTED_QTY * unit impact

Use **Documents_search**:
- Search for: "Supplier Security Incident Notification Procedure" (Doc 05)
- Extract escalation criteria and response time requirements

---

### STEP 3: GENERATE DOCUMENT

Use **code_execution** to build the report.

#### Document structure:
```
Page 1: Title Page
  - Report name, period, generated timestamp
  - Quality standard reference (ISO 9001 / internal QMS)

Page 2: Quality Scorecard
  - KPI tiles: overall rejection rate, yield, QC holds, open CAPAs
  - Traffic light indicators: GREEN / YELLOW / RED
  - Period-over-period trend arrows

Page 3: Pareto Analysis
  - Pareto chart: defect codes ranked by frequency (bar + cumulative line)
  - 80/20 line marked
  - Table: top 5 defect codes with description, frequency, and impacted materials

Page 4: Supplier / Product Quality Matrix
  - Heatmap or matrix: suppliers (rows) x metrics (columns) with color coding
  - Or: products (rows) x quality metrics (columns)
  - Highlight RED cells for immediate attention

Page 5: Trend Analysis
  - Line chart: rejection rate trend by month (with target line at 2%)
  - Line chart: yield trend (with target line at 95%/98%)
  - Annotate significant shifts or spikes

Page 6: Policy Compliance (for audit readiness)
  - Checklist format from Doc 07 criteria
  - Status: Compliant / Non-Compliant / Needs Review
  - Supporting evidence references

Page 7: Exception & Action Items
  - RED-tier suppliers: name, rejection rate, recommended action
  - Repeat defects: pattern, root cause hypothesis, CAPA recommendation
  - QC holds impacting availability: material, hold qty, saleable impact

Final Page: Data Sources & Methodology
  - Quality thresholds: GREEN < 2%, YELLOW 2-5%, RED > 5%
  - Semantic views and documents queried
  - Audit standard references
```

#### Quality-specific formatting:
- **Risk colors**: GREEN (#27AE60) < 2%, YELLOW (#F39C12) 2-5%, RED (#E74C3C) > 5%
- **Pareto chart**: Bars in descending order (navy), cumulative line (red), 80% line (dashed)
- **Heatmap**: White (best) to dark red (worst) gradient
- **Policy citations**: Italic with document name and section reference
- **Defect codes**: Monospace font, linked to description

---

### STEP 4: DELIVER

1. Save the file to the workspace.
2. Summarize:
   - Overall quality health: how many suppliers/products are GREEN/YELLOW/RED
   - Top defect: most frequent defect code and its impact
   - Critical alerts: any RED-tier suppliers or repeat defects
   - Policy gaps (for audit readiness reports)
3. Ask: "Would you like to drill into a specific supplier, defect type, or generate a CAPA action plan?"

---

### GUARDRAILS

- Rejection rates and defect data come ONLY from quality_analyst (SV_QUALITY_PERSONA).
- Manufacturing yield comes from manufacturing_analyst (SV_MANUFACTURING_PERSONA).
- Do NOT conflate incoming inspection rejection rate (supplier quality) with final test rejection rate (assembly quality). Always label which type.
- Policy content must come from Documents_search (Cortex Search). Never fabricate policy language.
- Always attribute policy citations: "Per [Document Name], Section [X]..."
- Quality thresholds (GREEN/YELLOW/RED) are per POL-RISK-004. State the thresholds explicitly.
