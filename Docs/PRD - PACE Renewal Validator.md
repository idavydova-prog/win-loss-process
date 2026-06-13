# PRD: PACE Renewal Validator — Org62 URL-Based Fetch & Comprehensive Validation

**Owner:** Ilona Davydova, CSM Leader, AMER PACE RCG  
**Status:** In Development  
**Date:** 2026-06-10  
**Last Updated:** 2026-06-10  
**Tool location:** `/Win-Loss/Skills/Renewal Validator.html` (local); GitHub Pages (shareable)

---

## Change Log

| Date | Change |
|------|--------|
| 2026-06-10 | Initial PRD created; plan approved |
| 2026-06-10 | Implemented tabbed UI (Fetch from Org62 / Paste Manually), gear icon settings panel, Slack token setup flow |
| 2026-06-10 | Implemented all Slack API functions: `openDMWithSlackbot`, `sendSlackMessage`, `pollForResponse`, `fetchFromSlackbot`, `fetchAndValidate` |
| 2026-06-10 | Refactored validation into shared `buildFindings()` used by both tabs |
| 2026-06-10 | **Parser fix:** Slackbot wraps long CSG Notes across indented continuation lines — parser now collects all indented lines as part of the same field value, preventing hashtags on line 2+ from being missed |
| 2026-06-10 | **Parser fix:** `Premier Attached: Yes` correctly recognized as truthy (already handled by `isYes()`; confirmed working) |
| 2026-06-10 | **Validation fix:** Attrition Reason comparison upgraded from character-overlap to word-level Jaccard similarity — prevents false "similar enough" matches between unrelated reasons (e.g. "Customer Relationship" vs "Financial & Contractual") |
| 2026-06-10 | **Validation fix:** Competitor and APM L1 empty-value detection now treats Slackbot placeholder strings (`[Not populated]`, `none`, `n/a`, `—`) as missing rather than populated |
| 2026-06-10 | Proof-of-concept Slackbot query tested with Opportunity ID `0063y000019usTFAAY`; confirmed fetch+parse flow works; workspace token setup pending IT admin approval for enterprise workspace |

---

## Overview

The PACE Renewal Validator is a standalone HTML tool that helps CSMs validate their renewal opportunity records against the PACE Attrition Hashtag Guidance. The current version requires manual copy-paste of Salesforce fields. This PRD defines the upgrade: CSMs enter an Org62 URL, the tool fetches all relevant data automatically via Slackbot, and runs a comprehensive compliance check with zero manual data entry.

---

## Problem Statement

CSMs managing PACE renewals must ensure their Salesforce records (CSG Notes hashtags, Red Account fields, financial posture) comply with team guidance before a deal moves forward. Today:

- There is no automated way to check compliance — CSMs do it manually or skip it
- Validation requires copying 10+ fields from Salesforce into a separate tool
- Red Account and Renewal fields must be checked for cross-record consistency, which is error-prone manually
- Non-compliant hashtags (wrong format, missing required tags, wrong field) reach the reporting layer and corrupt attrition dashboards

This leads to data quality issues, missed plays, and CSMs getting flagged during deal reviews for fixable formatting errors.

---

## Goals

| Goal | Measure |
|------|---------|
| Zero-friction validation | CSM inputs only one or two URLs; no copy-paste |
| Comprehensive compliance check | All PACE Attrition Hashtag Guidance rules enforced |
| Fully standalone | Single HTML file, no server, no Node.js, shareable to any CSM |
| Cross-record validation | Renewal ↔ Red Account consistency checked automatically |
| Actionable output | Each finding states what is wrong, what to fix, and why it matters |

---

## User Stories

**As a CSM**, I want to paste my renewal opportunity URL and get an instant compliance report so that I can fix issues before a deal review.

**As a CSM**, I want the tool to automatically fetch all renewal details from Org62 so that I don't have to manually copy fields out of Salesforce.

**As a CSM**, I want to see exactly which hashtags are missing, wrong, or in the wrong field so that I know precisely what to update.

**As a CSM leader**, I want all team members to use the same validator with no setup beyond a one-time token so that adoption is not blocked by technical barriers.

**As a CSM**, I want the tool to work on my local machine and be shareable via a file so that I don't need IT approval or a deployed server.

---

## Functional Requirements

### FR-1: URL Input Mode (new default)

- Input field for Renewal Opportunity URL (required)
- Input field for Red Account URL (optional)
- Org62 URL format: `https://org62.lightning.force.com/lightning/r/[Object]/[RecordId]/view`
- Tool extracts 15- or 18-character Salesforce record ID from the URL
- "Fetch from Org62" button triggers the fetch flow
- Status indicator shows progress: Connecting → Fetching renewal → Fetching Red Account → Validating → Done
- Errors shown inline (e.g., "Could not reach Slackbot — check your token in Settings")

### FR-2: Slackbot Fetch Flow

- Tool opens a DM with Slackbot via Slack Web API (`conversations.open` with `USLACKBOT`)
- Sends a natural-language read query for all PACE-relevant fields
- Polls DM history every 2 seconds for Slackbot's response (30-second timeout)
- Feeds raw Slackbot response into existing field parser (`buildFieldMap()`)
- If Red Account URL provided, same flow runs for the Red Account record
- No Slack window opens — all calls are invisible background API requests

**Renewal fields fetched:**
Opportunity Name, Account Name, Owner, Stage, Close Date, Forecasted Attrition, Forecasted Renewal % - New, Premier Attached, Targeted Clouds, License Renewal Status, Attrition Reason, Attrition Reason Detail, CSG Notes

**Red Account fields fetched:**
Account Name, Stage, ECOMM Headline, Latest Update, Closure Summary, Competitor (Primary), Competitor (Other), APM L1, APM L2, APM L3, Attrition Reason, Forecast Amount

### FR-3: Paste Manually Mode (existing, preserved)

- Tab 2 keeps the existing two-textarea layout exactly as-is
- Accepts Slackbot output (colon-separated format) or Salesforce Lightning copy-paste
- All existing pre-loaded examples (GNC Holdings, Wolverine World Wide) remain accessible here

### FR-4: One-Time Token Setup

- Gear icon (top-right of header) opens a settings panel
- Panel includes step-by-step instructions for obtaining a Slack user token (`xoxp-...`)
- Token input field (password-masked)
- "Save & Test" button calls Slack `auth.test` and shows "Connected as [Name]" or an error
- Token stored in browser `localStorage` under key `pace_slack_token`
- Token persists across sessions on the same browser — setup is done once

### FR-5: Renewal Opportunity Validation

**CSG Notes**
| Rule | Severity |
|------|----------|
| CSG Notes field is empty or < 10 characters | Critical |
| No date found in notes | Warning |
| No CSM name found (two words before first hashtag) | Warning |
| No license hashtag: missing `#LicNoAttrit`, `#LicPartialAttrit`, or `#LicFullAttrit` | Critical |
| Premier Attached = True but no `#SigRisk` or `#NoSigRisk` | Critical |
| Both Lic and Sig hashtags present but Lic comes after Sig (should be: `#LicX #SigX`) | Info |
| Red Account linked but `#RA` absent | Critical |
| `#RA` present with no name or URL following it | Info (suggest adding link) |
| `#LicPartialAttrit`, `#LicFullAttrit`, or `#SigRisk` present but no dollar estimate | Warning |
| Agentforce in scope but no `#AFAttrit` or `#AfNoRisk` | Warning |
| Data Cloud / D360 in scope but no `#D360Attrit` or `#D360NoRisk` | Warning |
| Tableau risk mentioned but no `#PBI` | Info |
| Invalid hashtag used (e.g. `#PartialAttrit`, `#FullAttrit`, `#NoAttrit`) | Critical |
| Unrecognized hashtag present | Info |
| Any single entry exceeds 255 characters | Warning |

**Financial / Status Cross-Checks**
| Rule | Severity |
|------|----------|
| License Renewal Status = "Will Reduce" or "Will Not Renew" but `#LicNoAttrit` present | Warning |
| License Renewal Status = "Will Renew in Full" but `#LicPartialAttrit` or `#LicFullAttrit` present | Warning |
| Full attrition scenario (0% renewal, $0 new ACV) but only `#LicPartialAttrit` tagged | Warning |

### FR-6: Red Account Validation

| Field | Rule | Severity |
|-------|------|----------|
| ECOMM Headline | Must be present if Red Account linked | Critical |
| ECOMM Headline | Must contain at least one offense hashtag (`#SIGTRIAL`, `#ARI`, `#SWAP`, `#IMPLEMENT`, `#ADOPT`, `#RFP`, `#REVIVE`) | Critical |
| ECOMM Headline | Defense hashtags present (`#RA`, `#LicX`, `#SigX`, etc.) — these belong in CSG Notes | Warning |
| ECOMM Headline | Hashtag in Headline but Latest Update says play was rejected/declined/not a fit | Warning |
| Latest Update | Missing or empty | Warning |
| Closure Summary | Red Account Stage = Closed/Resolved but Closure Summary is empty | Warning |
| Competitor | Primary and Other Competitor both empty | Info |
| APM L1 | Missing | Warning |
| APM L1 | Agentforce or Data Cloud at risk but APM L1 ≠ "AI & Data" | Warning |

### FR-7: Cross-Record Validation (Renewal ↔ Red Account)

| Rule | Severity |
|------|----------|
| Red Account URL entered but `#RA` absent from CSG Notes | Critical |
| Attrition Reason on Renewal and Red Account meaningfully differ | Warning |
| Red Account Stage = Precautionary but financial posture indicates full attrition | Warning |
| ECOMM Headline hashtag (e.g. `#ARI`) has no corresponding mention in CSG Notes | Info |
| AF or D360 attrition risk in notes exceeds $250K but no Red Account provided | Warning |
| AF/D360 risk ≤ $250K and no Red Account — hashtag is the primary signal | Info |
| Multi-cloud Red Account with AF/D360 > $250K — guidance recommends a separate Red for AF/DC (APM L1 = AI & Data) plus CrossCloud Red | Info |
| APM product detail mentioned in CSG Notes but no Red Account provided | Info (product detail should live on Red Account) |

### FR-8: Results Output

- Findings grouped by: Critical → Warning → Info
- Each finding includes: field name, what was found, what is expected, guidance reference
- "Pass" state shown when all critical and warning checks pass
- "Generate Email Summary" button (existing) remains functional
- Results section hidden until validation runs

---

## Non-Functional Requirements

| Requirement | Detail |
|-------------|--------|
| Standalone | Single `.html` file; no external dependencies beyond Slack Web API calls |
| No server required | All logic runs client-side in the browser |
| No Node.js | Not viable on team Macs; not required by this design |
| Shareable | File can be emailed, shared via Slack, or posted to GitHub Pages |
| Browser compatibility | Chrome and Safari on macOS (team standard) |
| Token security | Token stored only in user's own browser `localStorage`; never transmitted except to `api.slack.com` |
| Fallback mode | Manual paste tab preserved; tool is fully usable without a Slack token |
| Performance | Fetch + parse + validate completes within 45 seconds (Slackbot poll timeout); UI shows live status through each step |

---

## Out of Scope

- Writing back to Salesforce (read-only validation tool)
- OAuth server flow or any backend infrastructure
- Multi-user or shared state
- Salesforce API direct integration (no Org62 API credentials available in browser)
- Validation of fields outside the PACE Attrition Hashtag Guidance canvas
- Win/Loss story generation (separate tool in same project)
- Mobile or non-macOS browsers

---

## Guidance Reference

All validation rules derived from:  
**PACE Attrition Hashtag Guidance** — Slack Canvas, File ID: F0APZKWHHJ6, authored by Chris Barbee  
`https://salesforce.enterprise.slack.com/docs/T2E6RHTM0/F0APZKWHHJ6`
