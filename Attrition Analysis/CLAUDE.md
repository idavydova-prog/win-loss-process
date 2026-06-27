# PACE Attrition Analysis

## How to Run

**Command:** "Run the attrition analysis for [Cloud] for [Period]"

Examples:
- "Run the attrition analysis for MC for Q1"
- "Run the attrition analysis for Commerce Cloud for Q2 FY27"
- "Run the attrition analysis for Core for Q1"

## Query Template

```sql
SELECT Id, Account.Name, Forecasted_Attrition__c, StageName, Type,
       License_At_Risk_Reason__c, ACV_Reason_Detail__c, SEM_Notes__c, CloseDate
FROM Opportunity
WHERE Targeted_Clouds__c INCLUDES ('{CLOUD}')
  AND StageName IN ('05 Closed', 'Dead Attrition')
  AND Forecasted_Attrition__c < -100000
  AND Account.CSG_Region__c = 'AMER PACE'
  AND CloseDate >= {START} AND CloseDate <= {END}
ORDER BY Forecasted_Attrition__c ASC
```

## Cloud Parameter

| Cloud | INCLUDES value(s) |
|-------|-------------------|
| MC / Marketing Cloud | `'MC ExactTarget'` |
| CC / Commerce Cloud | `'Commerce Cloud'`, `'B2B Commerce'`, `'Order Management'` |
| Core | `'Core'` |
| Tableau | `'Tableau'` |
| Integration / MuleSoft | `'Integration Cloud'` |

For multi-value clouds (Commerce), use: `Targeted_Clouds__c INCLUDES ('Commerce Cloud', 'B2B Commerce', 'Order Management')`

## Period Parameter (Salesforce FY starts Feb 1)

### Quarterly

| Period | Start | End |
|--------|-------|-----|
| Q1 FY27 | 2026-02-01 | 2026-04-30 |
| Q2 FY27 | 2026-05-01 | 2026-07-31 |
| Q3 FY27 | 2026-08-01 | 2026-10-31 |
| Q4 FY27 | 2026-11-01 | 2027-01-31 |
| Q1 FY28 | 2027-02-01 | 2027-04-30 |

### Monthly

| Period | Start | End | Output filename |
|--------|-------|-----|-----------------|
| February FY27 | 2026-02-01 | 2026-02-28 | `[Cloud] Feb FY27.html` |
| March FY27 | 2026-03-01 | 2026-03-31 | `[Cloud] Mar FY27.html` |
| April FY27 | 2026-04-01 | 2026-04-30 | `[Cloud] Apr FY27.html` |
| May FY27 | 2026-05-01 | 2026-05-31 | `[Cloud] May FY27.html` |
| June FY27 | 2026-06-01 | 2026-06-30 | `[Cloud] Jun FY27.html` |
| July FY27 | 2026-07-01 | 2026-07-31 | `[Cloud] Jul FY27.html` |
| August FY27 | 2026-08-01 | 2026-08-31 | `[Cloud] Aug FY27.html` |
| September FY27 | 2026-09-01 | 2026-09-30 | `[Cloud] Sep FY27.html` |
| October FY27 | 2026-10-01 | 2026-10-31 | `[Cloud] Oct FY27.html` |
| November FY27 | 2026-11-01 | 2026-11-30 | `[Cloud] Nov FY27.html` |
| December FY27 | 2026-12-01 | 2026-12-31 | `[Cloud] Dec FY27.html` |
| January FY27 | 2027-01-01 | 2027-01-31 | `[Cloud] Jan FY27.html` |

**Monthly command:** "Run the attrition analysis for [Cloud] for [Month] FY27" (e.g., "Run the attrition analysis for MC for February FY27")

## Fixed Filters (do not change)

| Filter | Field | Value | Notes |
|--------|-------|-------|-------|
| Territory | `Account.CSG_Region__c` | `= 'AMER PACE'` | Consolidated value for RCG + CMRCL + ENTR MAE |
| Stage | `StageName` | `IN ('05 Closed', 'Dead Attrition')` | Both partial and full attrition |
| Threshold | `Forecasted_Attrition__c` | `< -100000` | >$100K attrition at opportunity header level |
| No SupportLevel filter | — | — | PACE includes Basic, Premier, Signature |
| No Type filter | — | — | Includes Renewal + all Off-Cycle variants |
| No RecordType filter | — | — | |
| No Owner Role filter | — | — | |

## Analysis Output (4 Leadership Questions)

After pulling data, produce analysis answering:

### 1. How much attrition was taken
- Total $ amount
- Dead Attrition (full logo loss) vs 05 Closed (partial — customer retained)
- On-Cycle (Type = 'Renewal') vs Off-Cycle (KMODs)
- Account count in each category

### 2. How much was saved & which programs
Scan `SEM_Notes__c` for program hashtags:
- `#ARI` — At-Risk Intervention
- `#SWAP` — Product Swap
- `#SIGTRIAL` — Signature Trial
- `#RFP` — Competitive Response
- `#REVIVE` — Revive
- `#SWE` — Strategic Win Engagement
- `#IMPLEMENT` — Implementation
- `#ADOPT` — Adoption

For each program found, list: account name, $ at risk, outcome (saved/lost/in progress)

### 3. Where were the gaps
- Accounts with no CSG Notes (`SEM_Notes__c` is null)
- Dead Attrition with no documented engagement
- Accounts with notes but no program hashtag (documented but no save play)
- Accounts tagged `#nonactionable`
- No attrition reason filled (`License_At_Risk_Reason__c` null or "Unspecified")
- No Red Account documentation (no `#RA` mention)

### 4. What patterns are emerging
- By `License_At_Risk_Reason__c` (reason category) — count + $ + %
- By `ACV_Reason_Detail__c` (specific reason) — top 5-10
- Competitor mentions in `SEM_Notes__c` (Klaviyo, Braze, Adobe, HubSpot, etc.)
- Dead vs Retained ratio
- Structural observations (saveable categories, repeat patterns)

## Key Field Reference

| Field | What it tells you |
|-------|-------------------|
| `Forecasted_Attrition__c` | Attrition amount (stored as negative number) |
| `StageName` | "Dead Attrition" = full logo loss; "05 Closed" = partial (customer retained) |
| `Type` | "Renewal" = on-cycle; "Off-Cycle..." variants = KMODs |
| `License_At_Risk_Reason__c` | Reason category (Financial & Contractual, Business Change & Distress, Product, etc.) |
| `ACV_Reason_Detail__c` | Specific reason (Low Perceived ROI, Downsizing, Feature Gap, etc.) |
| `SEM_Notes__c` | CSG Notes — program hashtags, competitor names, CSAL context |
| `Account.CSG_Region__c` | Territory — always 'AMER PACE' for this query |

## HTML Prototype Output

After analysis, generate a self-contained HTML prototype saved to `Prototypes/[Cloud] [Period].html` (e.g., `Prototypes/MC Q1 FY27.html`).

### Structure (4 tabs matching 4 questions)

| Tab | Title | Contents |
|-----|-------|----------|
| 1 | How Much Taken | Answer box (total $), 4 stat tiles (Logo Loss/Partial/On-Cycle/Off-Cycle), donut charts (split + cycle), top 10 horizontal bar chart, full accounts table |
| 2 | What Was Saved | 3 answer boxes (Confirmed Saves/Active Programs/No Program %), programs bar chart, program detail tables (ARI/RFP/SWE/etc.) with status badges, program scorecard |
| 3 | Where Are the Gaps | 3 stat tiles, gap severity bars, engagement funnel chart, gap composition donut, silent losses table, documented-but-no-program table |
| 4 | Emerging Patterns | Numbered insight boxes, reason breakdown horizontal bar, reason detail table, competitor cards, program pattern table, leadership action summary |

### Design Requirements

- **SLDS styling** (Salesforce Lightning Design System: light theme, brand tokens)
- **Chart.js CDN** loaded: `<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.7/dist/chart.umd.min.js"></script>`
- **Hero banner** at top: gradient navy background, total attrition $, account count, logos lost, retained, confirmed saves
- **Sticky nav** with numbered question tabs
- **Question banner** at top of each tab section (question number + full question text)
- **Bar chart spacing**: use `barPercentage: 0.6` + `categoryPercentage: 0.85` (not fixed barThickness) to prevent bars from merging
- **Color coding**: Dead/Logo Loss = red (#ea001e), Partial = orange (#fe9339), Programs = green (#2e844a), Info = blue (#0176d3)

### Accounts Table Requirements

The "All Accounts" table in Tab 1 must include:

| Column | Source | Display |
|--------|--------|---------|
| # | Row number | Sequential |
| Account | Account.Name | **Hyperlinked** to Org62 opportunity: `https://org62.lightning.force.com/lightning/r/Opportunity/[Id]/view` |
| Renewed | `Amount` field from Opportunity | Green if >$0, gray if $0 |
| Attrition | `Forecasted_Attrition__c` (absolute value) | Red, bold |
| Outcome | Based on StageName | Badge: Logo Loss (Dead Attrition) or Partial (05 Closed) |
| Type | `Type` field | Renewal or Off-Cycle |

### Program Hashtag Definitions

| Hashtag | Full Name |
|---------|-----------|
| #ARI | Attrition Reduction Investment |
| #RFP | Competitive Response (Request for Proposal) |
| #SWE | Strategic Win Engagement |
| #SWAP | Product Swap |
| #SIGTRIAL | Signature Trial |
| #IMPLEMENT | Active Implementation |
| #ADOPT | Adoption of Shelfware |
| #REVIVE | Top 100 Tableau Attrition Revive |

### Status Badges for Program Tables

- **Won** (green) — Save confirmed
- **Lost** (red) — Customer left despite program
- **In Progress** (blue) — Program active, outcome pending
- **Pending** (blue) — Awaiting approval
- **Declined** (purple) — Customer declined the program

## Validation Context

- Query validated 2026-06-24 against manual Tableau report "PACE MC Q1 Attrition >$100K Details"
- Query returns a superset of the manual report (28 vs 21 for MC Q1)
- Difference: Tableau computes attrition at MC line-item (APM) level; this query uses opportunity-header `Forecasted_Attrition__c`
- All 21 manual report accounts confirmed present in query results — zero false negatives

### One-Time Validation (per cloud)

**Source dashboard:** "Customer Success Metrics by Cloud — Attrition Drilldown" (Tableau)

**Manual steps (you do once per cloud):**
1. Open Tableau dashboard
2. Set filters: Cloud = [target], Attrition Event Month = [target month], Area = AMER PACE
3. Export to CSV
4. Drop file into `Validation/` folder

**Command:** "Validate [Cloud] [Month] FY27 against the Tableau export"

**Claude will:**
- Parse the Tableau CSV
- Match every Tableau account to SOQL results by Account Name
- Report: matches, SOQL-only (expected superset), Tableau-only (should be zero)
- Flag delta reason (APM-level threshold vs. opportunity-header threshold)

**Validation status:**
| Cloud | Validated | Result |
|-------|-----------|--------|
| MC | 2026-06-24 | 28 vs 21, zero false negatives |
| CC | Not yet | — |
| Core | Not yet | — |
| Tableau | Not yet | — |

## Dual Output (HTML + CSV for G-Sheet)

Each run produces two outputs from a single SOQL pull:
1. **HTML report** → `Prototypes/[Cloud] [Month] FY27.html` (leadership view)
2. **CSV extract** → `Data/[Cloud] [Month] FY27.csv` (G-sheet import for team)

**G-sheet import:** File → Import → Upload CSV → "Insert new sheet" or paste into existing tab. ~30 seconds per cloud/month. No separate manual process needed.

## Do NOT Rules (Lessons Learned)

### Nav Back Buttons — Use position: absolute
When adding a "← Home" back button to a sticky nav bar, do NOT use `margin-right: auto` (this pushes the tab buttons off-center). Instead use:
```css
.nav-back {
    position: absolute;
    left: 1.5rem;
    top: 50%;
    transform: translateY(-50%);
}
```
Note: `position: sticky` already acts as a containing block for absolutely-positioned children — do NOT add a redundant `position: relative` after it (last declaration wins and breaks the sticky).

### Chart.js Bar Settings
Use `barPercentage: 0.6` and `categoryPercentage: 0.85` for bar charts — prevents bars from being too wide or too narrow.

### Month View Ordering
When displaying monthly reports in a list/grid, order newest-first (e.g., Jul → Feb) so the most recent month is immediately visible without scrolling.
