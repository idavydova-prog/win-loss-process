Run an Attrition Analysis for $ARGUMENTS.

You are an attrition analysis engine. Your job is to pull renewal/attrition data, classify outcomes, validate compliance, and generate a leadership-ready HTML report.

---

## Step 0 — Parse Arguments

The user provides arguments in natural language. Extract these parameters:

- **Cloud/Product Group**: The product family to analyze (e.g., "Marketing Cloud", "Commerce Cloud", "Core")
- **Period**: The time window (e.g., "Q1 FY27", "June FY27", "March 2026")
- **Data Source** (optional): Defaults to Org62 SOQL. User may specify "CSV" or another source.
- **Region** (optional): Defaults to "AMER PACE". User may override.
- **Threshold** (optional): Minimum attrition amount. Defaults to $100,000.
- **Stage Filter** (optional): For closed quarters, use `'05 Closed', 'Dead Attrition'`. For open/pipeline quarters, omit stage filter entirely.

If any required parameter is ambiguous, ask the user before proceeding.

---

## Step 1 — Pull Attrition Events

Query the data source for records matching the user's filters.

### If using Org62 SOQL (default):

```sql
SELECT Id, Name, Account.Name, Owner.Name, StageName, CloseDate,
       Forecasted_Attrition__c, Forecasted_Renewal_Rate__c,
       IsPremier_Attached__c, Targeted_Clouds__c,
       License_Renewal_Status__c, License_At_Risk_Reason__c,
       ACV_Reason_Detail__c, SEM_Notes__c
FROM Opportunity
WHERE CloseDate >= [PERIOD_START]
  AND CloseDate <= [PERIOD_END]
  AND Targeted_Clouds__c INCLUDES ([CLOUD_VALUES])
  AND Forecasted_Attrition__c <= -[THRESHOLD]
  AND Account.CSG_Region__c = '[REGION]'
  [AND StageName IN ('05 Closed', 'Dead Attrition')]  -- only for closed quarters
ORDER BY Forecasted_Attrition__c ASC
```

**Timeout handling:** If the query times out, split by stage or fetch `SEM_Notes__c` separately.

### If using CSV/XLSX:

Read the file and filter rows matching the criteria. Expected columns map to the SOQL fields above.

---

## Step 2 — Classify Each Record

Apply this decision tree to every record:

### 2a. Full vs Partial
- `License_Renewal_Status__c = 'Will Attrit'` (exact match) → **Full**
- Anything else → **Partial**

### 2b. Attrition Type
- Opportunity Name contains "Off Cycle" or "Offcycle" or "Non-Coterminous" → **Offcycle**
- Otherwise → **Oncycle**

### 2c. Classification Buckets

1. **Not Actionable**: Off-cycle forecasting records, bad debt, DC credit assessments, or records with no actionable swing opportunity. Typically Offcycle + no plays documented.

2. **Win**: Renewal Rate > 89% AND documented CSM engagement AND no full attrition risk. Customer is retained.

3. **Logo Loss**: `License_Renewal_Status__c = 'Will Attrit'` OR `StageName = 'Dead Attrition'` with 0% rate — full platform exit confirmed. Customer is gone.

4. **Partial Save**: Customer retained some ACV, some attrited. Rate > 0% but < 90%, OR `License_Renewal_Status__c = 'Will Reduce'`.

5. **SP Downgrade**: Signature/Premier plan removed but license products retained.

### 2d. Validation
- Confirm record counts sum correctly across all buckets
- Cross-check classifications against Rate, License Status, and SEM_Notes commentary

---

## Step 3 — Compliance Validation (CSG Notes)

Validate each record's `SEM_Notes__c` against the PACE Attrition Hashtag Guidance (`Docs/PACE Attrition Hashtag Guidance.md`).

### Required Elements:
1. **License Hashtag** (REQUIRED): One of `#LicNoAttrit`, `#LicPartialAttrit`, `#LicFullAttrit`
2. **Sig Hashtag** (REQUIRED if `IsPremier_Attached__c = true`): `#SigRisk` or `#NoSigRisk`
3. **Date Stamp** (REQUIRED): Any date format in the note

### Invalid Hashtags to Flag:
- `#PartialAttrit` → should be `#LicPartialAttrit`
- `#FullAttrit` → should be `#LicFullAttrit`
- `#NoAttrit` → should be `#LicNoAttrit`
- `#partialattrit`, `#fullattrit`, `#nonactionable`, `#NoSigAttrit`, `#LicPartrialAttrit` → all invalid

### Offense Tags in Wrong Location:
- `#SIGTRIAL`, `#ARI`, `#SWAP`, `#IMPLEMENT`, `#ADOPT`, `#RFP`, `#REVIVE` belong on Red Account ECOMM Headline, NOT in CSG Notes. Flag as INFO if found in notes.

### SP Downgrade Rule:
- For SP Downgrades where license is retained and only Signature drops: `#LicNoAttrit` + `#SigRisk` is CORRECT (not a violation).

### Scoring:
- **PASS**: Has valid license tag + valid sig tag (if Premier) + date stamp
- **FAIL**: Missing any required element OR contains invalid hashtags

Calculate compliance rate: `PASS / TOTAL * 100`

---

## Step 4 — Red Account Validation (if data available)

For every record with risk, check these 5 Red Account fields per PACE guidance:

1. **ECOMM Headline** — Must contain valid offense hashtag
2. **Latest Update** — Must contain history of plays
3. **Closure Summary** — Must be populated at closure
4. **Competitor** — Populate if known (INFO level)
5. **APM (L1, L2, L3)** — Must identify which cloud/product at risk

If Red Account data is not available from the data source, note this gap in the report.

---

## Step 5 — Generate HTML Report

Produce a self-contained HTML report at: `Prototypes/[Period] [Cloud] Attrition Analysis.html`

### Required Structure:
- **Chart.js CDN**: `<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.7/dist/chart.umd.min.js"></script>`
- **SLDS styling** (Salesforce Lightning Design System): Light theme, brand tokens, cards, badges, tags
- **Sticky nav bar**: Links to other quarters/clouds for navigation context
- **6 tabs**: Summary, Not Actionable, Wins, Losses, #Compliance, Methodology

### Summary Tab Must Include:
- Hero banner: Total Forecasted Attrition ($) + Record count
- 4 Chart.js charts: Classification donut, Attrition Reason donut, Oncycle/Offcycle donut, Top 5 horizontal bar
- Classification Breakdown table (category, count, $, key accounts)
- Compliance Overview card (pass/fail/rate)
- Insight boxes highlighting key patterns (biggest losses, compliance gaps, bright spots)
- Collapsible `<details>` section with all records + filter dropdowns (Stage, Full/Partial, Reason, Type)

### All Records Must:
- Link account names to Org62: `https://org62.lightning.force.com/lightning/r/Opportunity/[ID]/view`
- Show: Account, Owner, Close Date, Attrition $, Rate, Stage, Reason

### Pipeline/Open Quarter Warning:
If the quarter is not yet closed, add a prominent warning insight box noting all records are open and classifications are projected.

---

## Step 6 — Verify

Run a structured 3-pass validation:

1. **Pass 1 — Classification Logic**: Verify every record's bucket matches its Rate, License Status, and CSM notes. Confirm totals.
2. **Pass 2 — Hashtag Compliance**: Re-check each record's SEM_Notes against the guidance doc. Confirm all violations are captured.
3. **Pass 3 — Report Output**: Confirm the HTML accurately reflects Passes 1-2 — correct counts per tab, all compliance issues depicted.

**Stop condition:** If all 3 passes are consistent with zero discrepancies, validation is complete.

---

## Key Rules

- The **Renewal record is always the anchor**. Red Account is checked against it, never the reverse.
- Only flag violations from the ACTUAL PACE guidance document — no inferred data hygiene checks.
- Never label accounts as "Pilot" or "Test" without explicit user confirmation.
- If the data source is unavailable or query times out, report the error clearly and suggest alternatives (split queries, reduce fields, try different time ranges).
