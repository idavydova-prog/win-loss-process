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

Cloud substitution:
| Cloud | INCLUDES values |
|---|---|
| Commerce Cloud | `'Commerce Cloud', 'B2B Commerce', 'Order Management'` |
| Marketing Cloud | `'MC ExactTarget', 'Marketing Cloud', 'Pardot', 'Datorama'` |

### Step 2 — Classify via Workflow Diagram

For each record:
1. **Risk?** Forecasted Attrition > $0 = Yes
2. **Actionable?** Swing > $0 = Actionable; Swing = $0 = Not Actionable
3. **Outcome:**
   - Win = $0 Attrit FINAL
   - Loss → sub-classify:
     - **Logo Loss** = Prior ACV fully attrited, customer gone
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
- 6 tabs: Summary, Not Actionable, Wins, Losses, #Compliance, Methodology
- Summary metrics (total attrition, classification counts, compliance rate)
- Records grouped by classification with Org62 links
- Red Account validation insight boxes (all 5 fields)
- CSG Notes compliance table with pass/fail per record
- AF/D360 decision tree findings where applicable

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
