Run the Win/Loss Process for $ARGUMENTS.

Execute all 5 steps of the Win/Loss Process:

1. **Pull** — Query Org62 via SOQL for attrition events matching the specified cloud and period (threshold >= $100K)
2. **Classify** — Apply the Win/Loss Workflow Diagram (Risk → Actionable → Outcome)
3. **Red Account Validation** — Validate all 5 PACE fields (ECOMM Headline, Latest Update, Closure Summary, Competitor, APM)
4. **CSG Notes Compliance** — Validate hashtags against `Docs/PACE Attrition Hashtag Guidance.md`
5. **Generate HTML Report** — Output to `Prototypes/[Period] [Cloud] Attrition Analysis.html`

After generating the report, validate all found accounts against the PACE guidance document at least 3 times to confirm accuracy.

Cloud substitution:
- Commerce Cloud → `'Commerce Cloud', 'B2B Commerce', 'Order Management'`
- Marketing Cloud → `'MC ExactTarget', 'Marketing Cloud', 'Pardot', 'Datorama'`

The user will provide arguments like: "Commerce Cloud for June FY27" or "Marketing Cloud for Q2 FY27"
