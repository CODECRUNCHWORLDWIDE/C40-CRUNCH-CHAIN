# Mini-Project — Spend Cube, Supplier Scorecard, and a Defended Sourcing Recommendation

> Build the full analytical stack this week taught, in order — spend cube, then scorecard, then lead-time-risk quantification — and use all three together to make one concrete sourcing recommendation for Crunch Gear's procurement leadership, backed by numbers at every step.

**Estimated time:** 3 hours, best done Saturday after the exercises and challenges.

This is the week's capstone: prove you can do the whole procurement-analytics workflow end to end, on your own, using the same `purchase_orders` seed table from this week's [README](../README.md) — not a new dataset. Real procurement analysts rarely get asked for just a spend cube, or just a scorecard, in isolation — they get asked "where should we focus, and what should we do about it," and have to assemble the full picture themselves.

---

## Deliverable

A directory in your portfolio `c40-week-06/mini-project/` containing:

1. `spend-cube.sql` **or** `spend_cube.py` — the full spend cube (category, supplier, business unit, month), Pareto/cumulative analysis, and tail-spend identification.
2. `scorecard.py` **or** `scorecard.sql` — a weighted supplier scorecard covering **all suppliers in one category of your choice** (Fabric or CMT — pick whichever you didn't already fully score in this week's exercises, to force yourself to compute it fresh).
3. `lead-time-risk.py` **or** `.sql` — a safety-stock/carrying-cost comparison (Lecture 3's method) between the two highest-volume suppliers in your chosen category.
4. `report.md` — a written recommendation memo, 500–700 words, that ties all three analyses together (structure below).

---

## Part A — Spend cube (45 min)

Using `purchase_orders`, produce:

1. Total Q1 spend, and spend broken out by category, by business unit, and by month.
2. A supplier-level Pareto/cumulative-percentage table (Lecture 1, Section 4) — identify how many suppliers make up 80% of spend.
3. A tail-spend / maverick-buying cut (Lecture 1, Section 5) — total maverick spend, as a percent of total spend *and* as a percent of its own category.

State, in one or two sentences per finding, what each number means for a procurement leader reading it for the first time.

## Part B — Supplier scorecard (60–75 min)

Pick **one category** you have not already fully scored in this week's exercises (if you did Fabric in Exercise 2, score CMT here, or vice versa — Trims is also a valid choice if you want an extra challenge with only two suppliers).

1. Compute the four raw metrics per supplier (volume-weighted price, defect rate, OTD %, lead-time standard deviation) — Lecture 2, Section 2.
2. Normalize each to 0–100 and combine into a weighted score using Lecture 2's default weights (30% cost / 30% quality / 25% OTD / 15% lead-time reliability) — Lecture 2, Sections 3–4.
3. Report each supplier's `n_orders` alongside its score, and flag (in a comment or a column) any supplier whose sample size is small enough that you'd want more data before acting on their score (Lecture 2, Section 5).

## Part C — Lead-time risk (45 min)

Take the **two highest-volume suppliers** from your Part B category and, using Lecture 3's method:

1. Compute each supplier's safety-stock requirement using `SS = z × daily_demand × σ_lead_time`, with **z = 1.65** (95% service level) and a daily-demand assumption you choose and justify in `report.md` (state your reasoning — e.g., based on that category's share of total Outerwear/Accessories unit volume).
2. Convert the safety-stock difference between the two suppliers into an annual carrying-cost dollar figure, using a **25% annual carrying cost rate**.
3. Build each supplier's full TCO per unit (unit price + freight/unit + quality cost/unit + carrying cost/unit), using a **$18 rework cost per defective unit** assumption, same as Lecture 3.

---

## Tying it together — `report.md`

Structure your memo in four sections:

1. **Spend overview** (from Part A) — where the money goes, and where the risk/opportunity concentration is.
2. **Supplier scorecard results** (from Part B) — who's actually performing well, with the small-sample caveat stated explicitly wherever it applies.
3. **Lead-time risk finding** (from Part C) — the dollar cost of choosing the lower-TCO supplier's lead-time variability over the higher-TCO/more-consistent one, or vice versa.
4. **Recommendation** — one concrete, numbered sourcing action (e.g., "consolidate X% of Trims spend from Zephyr onto ButtonWorks," "cap Supplier Y's award at Z units pending a quality corrective-action plan," "renegotiate Supplier W's contract given a $N unfavorable price variance this quarter"). Your recommendation must reference at least two of the three analyses (spend cube, scorecard, TCO/lead-time) as supporting evidence — a recommendation based on only one lens is incomplete.

---

## Rules

- All analysis in **SQL and/or Python** — no spreadsheet formulas, per this course's data rule (state which tool(s) you used in `report.md`).
- Show your queries/code, not just output — `spend-cube.sql`/`.py`, `scorecard.py`/`.sql`, and `lead-time-risk.py`/`.sql` are all required deliverables.
- Every number in `report.md` must be traceable to a query/code cell in one of your three analysis files.
- State every assumption you make (daily demand, rework cost, carrying-cost rate) explicitly — a number without its assumption stated is not a defensible number.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Spend cube — correctness & completeness | 20% | Category/BU/month/Pareto all correct and verified against the seed data |
| Scorecard — method | 20% | Correct normalization direction, correct weighting, sample size reported and flagged where thin |
| Lead-time risk / TCO — correctness | 20% | Safety-stock formula applied correctly; carrying cost and full TCO computed and traceable |
| Recommendation quality | 25% | Specific, numbered, cites at least two of the three analyses, states its own cost/trade-off |
| Assumption discipline | 10% | Every assumption (demand, rework cost, carry rate) stated explicitly, not buried or implicit |
| Tooling discipline | 5% | SQL/Python only, work shown, no spreadsheet formulas |

---

## Why this matters

This is the shape of a real procurement-analytics assignment: nobody hands you a pre-built spend cube *and* a pre-built scorecard *and* a TCO comparison — you build all three from the same raw transactional data, because that's what actually exists in a company's systems, and you're expected to synthesize them into one recommendation a VP can act on this quarter. Keep every file — Week 7 shifts from *supply-side* uncertainty (this week's supplier lead-time variability) to *demand-side* uncertainty, and the safety-stock formula you just used for lead-time risk reappears, pointed at forecast error instead.

When done: push, then take the [quiz](../quiz.md) and move on to Week 7.
