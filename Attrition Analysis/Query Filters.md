# PACE Attrition Query Filters

## SOQL Query Template

```sql
SELECT Id, Account.Name, Forecasted_Attrition__c, Amount, StageName, Type,
       License_At_Risk_Reason__c, ACV_Reason_Detail__c, SEM_Notes__c, CloseDate
FROM Opportunity
WHERE Targeted_Clouds__c INCLUDES ('{CLOUD_VALUES}')
  AND StageName IN ('05 Closed', 'Dead Attrition')
  AND Forecasted_Attrition__c < -100000
  AND Account.CSG_Region__c = 'AMER PACE'
  AND CloseDate >= {PERIOD_START} AND CloseDate <= {PERIOD_END}
ORDER BY Forecasted_Attrition__c ASC
```

---

## Cloud Parameter

Swap `{CLOUD_VALUES}` based on cloud:

| Cloud | INCLUDES value(s) |
|-------|-------------------|
| Marketing Cloud (MC) | `'MC ExactTarget'` |
| Commerce Cloud (CC) | `'Commerce Cloud'`, `'B2B Commerce'`, `'Order Management'` |
| Core | `'Core'` |
| Tableau | `'Tableau'` |
| Integration / MuleSoft | `'Integration Cloud'` |

For multi-value clouds (Commerce), use:
```sql
Targeted_Clouds__c INCLUDES ('Commerce Cloud', 'B2B Commerce', 'Order Management')
```

---

## Period Parameter

Salesforce Fiscal Year starts **February 1**.

| Period | Start Date | End Date |
|--------|-----------|----------|
| Q1 FY27 | 2026-02-01 | 2026-04-30 |
| Q2 FY27 | 2026-05-01 | 2026-07-31 |
| Q3 FY27 | 2026-08-01 | 2026-10-31 |
| Q4 FY27 | 2026-11-01 | 2027-01-31 |
| Q1 FY28 | 2027-02-01 | 2027-04-30 |

---

## Fixed Filters (Always Applied)

| # | Filter | Field | Value | Why |
|---|--------|-------|-------|-----|
| 1 | Territory | `Account.CSG_Region__c` | `= 'AMER PACE'` | Consolidated value covering RCG + CMRCL + ENTR MAE |
| 2 | Stage | `StageName` | `IN ('05 Closed', 'Dead Attrition')` | 05 Closed = partial (customer retained); Dead = full logo loss |
| 3 | Threshold | `Forecasted_Attrition__c` | `< -100000` | Stored as negative number; this filters to >$100K attrition |
| 4 | Cloud | `Targeted_Clouds__c` | `INCLUDES (...)` | Multipicklist — must use INCLUDES, not LIKE |
| 5 | Period | `CloseDate` | Date range | Fiscal quarter boundaries |

---

## What Is NOT Filtered (Intentionally)

| Filter | Why Not Applied |
|--------|----------------|
| SupportLevel | PACE includes Basic, Premier, and Signature — no tier exclusion |
| Opportunity Type | Includes Renewal + all Off-Cycle variants (Bad Debt, Restructure, KMOD) |
| RecordType | Not needed |
| Owner Role | Not needed |

---

## Key Fields Returned

| Field | What It Tells You |
|-------|-------------------|
| `Id` | Opportunity ID — used for Org62 hyperlinks |
| `Account.Name` | Customer name |
| `Amount` | Renewed amount (what the customer is paying going forward) |
| `Forecasted_Attrition__c` | Attrition amount (negative = attrition taken) |
| `StageName` | "Dead Attrition" = full logo loss; "05 Closed" = partial (retained) |
| `Type` | "Renewal" = on-cycle; "Off-Cycle..." variants = KMODs |
| `License_At_Risk_Reason__c` | Reason category (e.g., "Financial & Contractual") |
| `ACV_Reason_Detail__c` | Specific reason (e.g., "Low Perceived ROI") |
| `SEM_Notes__c` | CSG Notes — contains program hashtags and CSAL context |
| `CloseDate` | When the opportunity closed |

---

## Important Notes

- `Forecasted_Attrition__c` is at the **opportunity header level** (total across all products), not cloud-specific line-item level
- Tableau reports may show fewer accounts because they compute cloud-specific attrition at the APM line-item level (requires Prior ACV not available via SOQL)
- `Targeted_Clouds__c` is a multipicklist — always use `INCLUDES`, never `LIKE` or `=`
- `Account.CSG_Region__c = 'AMER PACE'` replaces the deprecated values: `'AMER ICE ENTR RCG'`, `'AMER ICE CMRCL'`, `'AMER ICE ENTR MAE'`
