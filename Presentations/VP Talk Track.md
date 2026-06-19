# Win/Loss Process Automation — VP Talk Track

## Opening (30 seconds)

"I want to show you something I've been building that solves a problem we've been living with for a while — the manual attrition reporting cycle. Every quarter, we pull data from Org62, classify outcomes, check compliance, and build slide decks for leadership. It takes hours, it's inconsistent across leaders, and we're missing coaching signals buried in the data. I've automated the entire thing."

---

## The Problem (1 minute)

"Here's what the process looks like today:

- A leader pulls SOQL data from Org62 — that's 20-30 minutes if the query doesn't time out
- They open a spreadsheet, manually classify each record as a Win, Loss, Partial Save, or Not Actionable — another 30 minutes, and two people might classify the same record differently
- They check CSG Notes compliance against the PACE guidance — this is where most people skip steps because it's tedious
- They build a report or slide deck — another hour or more
- Multiply that by 9 reports per quarter — 3 clouds, 3 quarters — and we're spending 20-30+ hours on reporting instead of coaching

The bigger issue isn't the time — it's what we're NOT seeing. In our pilot, we found that only 12% of Marketing Cloud records in Q1 had compliant CSG Notes. 38% were completely empty. That's a coaching gap we didn't know existed because nobody had time to check every record."

---

## The Solution (1-2 minutes)

"I built a workflow that does the entire 5-step process end-to-end. Any leader on the team can run it by typing one sentence: 'Run the Win/Loss Process for Marketing Cloud for Q1 FY27.'

It pulls the data, classifies every record using the same decision tree, validates compliance against our PACE guidance document, and generates an interactive HTML report with charts, drill-downs, and Org62 links.

Let me show you what the output looks like."

*[Open the dashboard or one of the reports in browser]*

"Each report has 6 tabs:
- **Summary** gives you the full picture — total attrition, classification breakdown, charts, and insight boxes that highlight the biggest patterns
- **Not Actionable, Wins, Losses** — each classification with detail tables and account-level Org62 links
- **Compliance** — this is the coaching tab. Every record scored pass/fail with specific violations called out
- **Methodology** — documents exactly how the data was pulled and classified, so there's no ambiguity

The classification logic is standardized. It doesn't matter who runs it — the same record always lands in the same bucket. That eliminates the 'it depends who's doing it' problem."

---

## Impact (1 minute)

"I ran the pilot across all 9 quarterly reports — Core, Commerce, and Marketing Cloud for Q1 through Q3 FY27. Here's what we got:

- **~30 hours of analyst time returned to strategic work** — that's per quarter, ongoing
- **100% classification consistency** — no more debates about whether something is a Logo Loss or a Partial Save
- **Compliance visibility we never had before** — we now know the exact rate, the exact accounts, and the exact CSMs with gaps
- **Immediate coaching signals** — the 12% compliance rate in MC Q1 told me exactly which 13 CSMs need a conversation about CSG Notes hygiene

And the reports are live, shareable links — not static slides that go stale. Leadership can click into Org62 directly from any account in the report."

---

## How We Measure Success (30 seconds)

"I'm proposing we track 4 things:

1. **Compliance rate** — baseline is 12% for MC. Target: 60% by Q4 as coaching takes effect
2. **Time-to-report** — from hours to under 15 minutes
3. **Empty CSG Notes** — baseline is 38%. Target: under 10%
4. **Coverage** — every cloud, every quarter, every cycle. No more gaps.

These aren't vanity metrics — compliance rate directly correlates to data quality in our dashboards and the accuracy of our attrition stories to leadership."

---

## The Ask (30 seconds)

"I'd like to roll this out as the standard monthly process for PACE AMER. What that means:

- Every leader uses this workflow to generate their quarterly reports
- We use the Compliance tab as a coaching input in 1:1s
- We track compliance improvement over time as a team health metric

It's already built, it's already been piloted on real data, and any leader can run it today. I just need the green light to make it official."

---

## Anticipated Questions

**"What if the classification logic is wrong for an edge case?"**
> "The methodology tab documents every rule. If we find an edge case, we update the decision tree centrally and it applies to all future runs. That's actually better than today, where each leader invents their own interpretation."

**"Who maintains this?"**
> "The workflow is code-based — it lives in a GitHub repo. I maintain it. Updates to PACE guidance get reflected once, and all future reports inherit the change."

**"Can other regions use it?"**
> "Yes — the region, cloud, threshold, and period are all configurable. EMEA or APJ could run it by changing one parameter."

**"What about Red Account validation?"**
> "That's the next phase. The workflow already has the framework — we're pending API access to all 5 Red Account fields. Once that's in, the report will surface Red Account gaps alongside compliance."

**"How do we know the data is accurate?"**
> "Every run includes a 3-pass verification step — classification logic, compliance scoring, and report output are each validated independently. The Methodology tab shows exact SOQL filters so anyone can reproduce the pull."
