# Win/Loss Process — Project Instructions

This folder contains the Win/Loss attrition analysis workflow for PACE AMER. When a user opens this folder in Claude Code, these instructions provide the context needed to run the process.

## Prerequisites

- **Org62 MCP Server** must be connected (provides SOQL access to Salesforce Org62)
- The PACE Attrition Hashtag Guidance lives at `Docs/PACE Attrition Hashtag Guidance.md`
- Output reports go to `Prototypes/[Period] [Cloud] Attrition Analysis.html`

## How to Invoke

Say: **"Run the Win/Loss Process for [Cloud] for [Period]"**

Examples:
- "Run the Win/Loss Process for Commerce Cloud for June FY27"
- "Run the Win/Loss Process for Marketing Cloud for Q2 FY27"

## The 5-Step Process

### Step 1 — Pull Attrition Events (SOQL)

Query Org62 for renewal opportunities matching:
- `CloseDate` in the target month/quarter
- `Targeted_Clouds__c INCLUDES (...)` for the target cloud
- `Forecasted_Attrition__c <= -100000` (attrition >= $100K)
- `StageName IN ('05 Closed', 'Dead Attrition')` for closed quarters (Q1/Q2); omit StageName filter entirely for open/pipeline quarters (Q3+)

Cloud substitution:
| Cloud | INCLUDES values |
|---|---|
| Core | `'Core'` |
| Commerce Cloud | `'Commerce Cloud', 'B2B Commerce', 'Order Management'` |
| Marketing Cloud | `'MC ExactTarget', 'Marketing Cloud', 'Pardot', 'Datorama'` |

Additional fields to include in SOQL:
- `Account.CSG_Subregion__c` (for Area segmentation)
- `Account.CSG_Portfolio__c` (for Portfolio segmentation)
- `Account.CSG_Region__c = 'AMER PACE'` (filter)
- For large result sets or timeout-prone queries, split into two queries (one per stage, or by month) and combine results

**Scaling:** When running for "all clouds for Q1–Q4", launch agents in parallel (3 clouds × N quarters). Each agent runs the full 5-step process independently.

**Salesforce Fiscal Year:** FY starts Feb 1. Q1 = Feb–Apr, Q2 = May–Jul, Q3 = Aug–Oct, Q4 = Nov–Jan.

### Step 2 — Classify via Workflow Diagram

For each record:
1. **Risk?** Forecasted Attrition > $0 = Yes
2. **Actionable?** Swing > $0 = Actionable; Swing = $0 = Not Actionable
3. **Outcome:**
   - Win = $0 Attrit FINAL
   - Loss → sub-classify:
     - **Logo Loss** = Prior ACV fully attrited, customer gone. Records with `StageName = 'Dead Attrition'` are confirmed losses — classify as Logo Loss unless they clearly retained some ACV.
     - **Partial Save** = Some ACV retained, some attrited
     - **SP Downgrade** = Signature/Premier removed, license retained

### Step 3 — Red Account Validation

For every record with Risk, validate ALL 5 Red Account fields per PACE guidance:

1. **ECOMM Headline** — Must contain valid offense hashtag: #SIGTRIAL, #ARI, #SWAP, #IMPLEMENT, #ADOPT, #RFP, #REVIVE. Flag invalid (#PRECAUTIONARY, #DCCAP, etc.) and defense tags in wrong location.
2. **Latest Update** — Must contain history of plays. Flag empty.
3. **Closure Summary** — Must be populated at closure (Lost/Resolved). Flag empty/minimal.
4. **Competitor** — Populate if known. INFO level.
5. **APM (L1, L2, L3)** — Must identify which cloud/product at risk. For AF/DC: APM L1 = "AI and Data".

### Step 4 — CSG Notes Compliance

Validate each record's `SEM_Notes__c` against PACE guidance:
- Must have license hashtag: #LicNoAttrit, #LicPartialAttrit, or #LicFullAttrit
- Must have sig hashtag if Premier: #SigRisk or #NoSigRisk
- Must have date stamp
- Flag known-invalid hashtags (#PartAttrit, #FullAttrit, #NoAttrit, #fullattrit, #nonactionable, etc.)

**SP Downgrade tagging principle:** The license hashtag tracks LICENSE/PRODUCT attrition only. The sig hashtag tracks Signature attrition separately. For SP Downgrades where license is retained and only Signature drops: #LicNoAttrit (no product attriting) + #SigRisk (Signature at risk) is CORRECT.

### Step 5 — Generate Leadership Report (HTML)

Produce a self-contained HTML report with:
- **Chart.js CDN** loaded before `<style>`: `<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.7/dist/chart.umd.min.js"></script>`
- **SLDS styling** (Salesforce Lightning Design System: light theme, brand tokens, cards, badges, tags)
- **Sticky nav bar** with links to other quarters (Q1–Q4) and clouds for the same period
- **6 tabs:** Summary, Not Actionable, Wins, Losses, #Compliance, Methodology
- **Summary panel:**
  - Hero banner: Total Forecasted Attrition ($ large font) + Record count
  - 5 Chart.js charts: Classification donut, Attrition Reason donut, Oncycle/Offcycle donut, Top 5 Accounts horizontal bar, **Area donut** (RCG/MAE/CMRCL)
  - Classification Breakdown table (category, count, attrition $, key accounts)
  - **Area Breakdown table** (area, records, attrition $, % of total)
  - Compliance Overview card (pass/fail/rate)
  - Insight boxes highlighting key patterns (including area distribution insight)
  - Collapsible Attrition Events section (`<details>`) with filter dropdowns (Event Month, Full/Partial, Attrition Reason, **Area**, Type) and table columns: Account Name (Org62 hyperlink), **Area**, Event Month, Full/Partial, Attrition Reason, Attrition Reason Detail, Attrition Type, Attrition Amount
- All account names hyperlinked to Org62: `https://org62.lightning.force.com/lightning/r/Opportunity/[ID]/view`
- Records grouped by classification with Org62 links
- Red Account validation insight boxes (all 5 fields)
- CSG Notes compliance table with pass/fail per record
- AF/D360 decision tree findings where applicable
- For pipeline/open quarters (Q3+): add a warning insight box noting all records are open and classifications are projected
- **Methodology tab** must include a **Success Metrics & Impact Tracking** section with these KPIs:

| Metric | Baseline (Q1 FY27) | Target | How Measured |
|---|---|---|---|
| CSG Notes Compliance Rate | 12% (MC) | ≥60% | #Compliance tab pass/fail per record |
| Time-to-Report | 2–4 hours / report | <15 minutes | End-to-end automation runtime |
| Empty CSG Notes (%) | 38% (MC Q1) | <10% | Records with no SEM_Notes__c |
| Classification Consistency | Varies by analyst | 0 reclassifications | 3-pass verification step |
| Report Coverage | Ad-hoc, incomplete | All clouds, all quarters | Report generation log |

  Include the current report's actual compliance rate and empty notes % in the "Current" column. Add quarter-over-quarter trend commentary and coaching mechanism explanation.

**Classification logic for Attrition Events table:**
- **Full/Partial:** `License_Renewal_Status__c = 'Will Attrit'` (exact match) → Full; everything else → Partial
- **Attrition Type:** Opportunity Name contains "Off Cycle" or "Offcycle" or "Non-Coterminous" → Offcycle; otherwise Oncycle

**Area segmentation:**
- `Account.CSG_Subregion__c` → Area: "AMER PACE MAE ENTR" → MAE, "AMER PACE RCG" → RCG, "AMER PACE CMRCL" → CMRCL
- Area colors: RCG = #0176d3 (blue), MAE = #7e57c2 (purple), CMRCL = #fe9339 (orange)
- Tag classes: `.tag-rcg{background:#eaf5fe;color:#0176d3}`, `.tag-mae{background:#ede7f6;color:#7e57c2}`, `.tag-cmrcl{background:#fff8e6;color:#946f00}`
- `Account.CSG_Portfolio__c` → Portfolio (sub-segmentation within Area)

Output: `Prototypes/[Period] [Cloud] Attrition Analysis.html`

## Validation

After generating the report, run a single structured validation with 3 distinct passes. Each pass checks something different:

**Pass 1 — Classification Logic:**
Verify every record's bucket (Win / Partial Save / Logo Loss / SP Downgrade / Not Actionable) against its Rate, ACV, License_Renewal_Status, and CSM commentary. Confirm the total sums to the record count.

**Pass 2 — Hashtag Compliance:**
Check each record's `SEM_Notes__c` against the valid tag lists in `Docs/PACE Attrition Hashtag Guidance.md`. Flag: invalid hashtags, empty CSG Notes, missing license tags, missing sig tags (Premier=true), offense tags in wrong field.

**Pass 3 — Report Output Verification:**
Confirm the generated HTML report accurately reflects all findings from Passes 1–2: correct record counts per tab, all compliance issues depicted in the #Compliance tab, and no findings omitted.

**Stop condition:** If all 3 passes produce consistent results with zero discrepancies, validation is complete. Do not repeat.

## Key Rules

- The **Renewal record is always the anchor**. Red Account is checked against it, never the reverse.
- Only flag violations from the ACTUAL PACE guidance document — no inferred data hygiene checks.
- Never label accounts as "Pilot" or "Test" without explicit confirmation.
- AF/D360 Decision Tree: "Is there AF or DC attrition risk? Yes → tag #AfAttrit / #D360Attrit (always, no exceptions)." This is NOT a Global Standard requirement but should be flagged as a gap.

## Report Inventory

Reports are published to GitHub Pages at: `https://idavydova-prog.github.io/win-loss-process/Prototypes/`

| Quarter | Period | Core | Commerce | Marketing |
|---|---|---|---|---|
| Q1 FY27 | Feb–Apr 2026 | ✅ Closed | ✅ Closed | ✅ Closed |
| Q2 FY27 | May–Jul 2026 | ✅ Closed | ✅ Closed | ✅ Closed |
| Q3 FY27 | Aug–Oct 2026 | ✅ Pipeline | ✅ Pipeline | ✅ Pipeline |
| Q4 FY27 | Nov–Jan 2027 | ✅ Pipeline | ✅ Pipeline | ✅ Pipeline |

All 16 reports include Area/Portfolio segmentation and Success Metrics tracking in the Methodology tab.

## Dashboard

The central hub is `Prototypes/dashboard.html` — links to all reports with summary stats (record count, attrition $, compliance rate). Update the dashboard whenever new reports are generated or existing ones are refreshed.
