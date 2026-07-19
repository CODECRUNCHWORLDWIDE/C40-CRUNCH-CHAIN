# Mini-Project — Run One Full S&OP Cycle

> Run one complete monthly S&OP cycle for Crunch Gear, across all three product families, six months, start to finish: demand review, supply review, a constrained-but-feasible production plan, and a financial reconciliation — packaged as the one-page executive summary a real leadership team would actually read at sign-off.

**Estimated time:** 2.5–3 hours, best done Saturday after the exercises and challenges.

This is the week's capstone, and it deliberately pulls together every piece: Exercise 1's balance table, Exercise/Challenge 1's gap-closing plan, and Lecture 3's financial reconciliation, run across **all three families** instead of just Trail Footwear. A real S&OP analyst doesn't get to solve one family in isolation and call it done — the executive summary has to say something true and useful about the whole portfolio in one page.

---

## Deliverable

A directory in your portfolio `c40-week-10/mini-project/` containing:

1. `balance-tables.sql` (or `.sql` + `.py`) — the full six-month, three-family supply-demand balance table, **plus** your recommended production plan (regular + any overtime/subcontract needed) that keeps every family above its safety-stock target every month.
2. `reconciliation.sql` (or `.py`) — your recommended plan's units converted to revenue per family per month, compared against `financial_targets`, with the variance in both dollars and percent.
3. `exec-summary.md` — a **one-page** written summary in the format specified below, the actual sign-off artifact.
4. `notes.md` — a short reflection (see the end).

Everything runs against the four seed tables from the [week README](../README.md). Works on PostgreSQL or SQLite; note which you used.

---

## The work, in four milestones

### Milestone 1 — Demand + supply review, all three families (45 min)

Build the regular-capacity-only balance table (Exercise 1's method) for **all three families**, not just Trail Footwear. Confirm which families/months breach the 900/700/1200 safety-stock targets respectively. *(You already know the answer for Trail Footwear from Exercise 1 — confirm Backpacks & Bags and Apparel independently rather than assuming.)*

### Milestone 2 — Close every real gap (60 min)

For any family/month that breaches its target using regular capacity alone, build a production plan (using overtime and/or subcontract, per `supply_plan`'s monthly ceilings) that closes it — reuse your [Challenge 1](../challenges/challenge-01-constrained-sop-plan.md) work for Trail Footwear if you completed it; if not, build a feasible (not necessarily cost-optimal) plan now. Verify with a query: every family, every month, ending inventory `>= safety_stock_target`.

### Milestone 3 — Reconcile against finance (45 min)

For every family and month, compute `achieved_units = LEAST(your_planned_supply, consensus_forecast_units)` and `achieved_revenue = achieved_units * unit_price`. Compare against `financial_targets.revenue_target`: compute the dollar variance and percent variance, per family per month, and as a **six-month, three-family grand total**.

*Spot check: Backpacks & Bags' six-month revenue, fully executed against its consensus forecast, should total close to $4.37M — use this to sanity-check your revenue formula before trusting the grand total.*

### Milestone 4 — Write the executive summary (30-45 min)

`exec-summary.md`, one page, in this exact structure (a real S&OP deck follows something close to this shape every month):

```markdown
# Trail Footwear / Backpacks & Bags / Apparel — S&OP Cycle Summary — [your name] — Jan-Jun 2025

## Headline
[One or two sentences: does the reconciled plan hit the six-month revenue target overall? By how much, in dollars and percent?]

## By family
[One line per family: on track / needs action, with the key number.]

## Risks & actions taken
[For each family/month that needed overtime or subcontract to stay feasible: what was
done, and what it cost.]

## Recommendation for sign-off
[Approve as-is / approve with a named condition / send back for rework — pick one,
and justify it in 2-3 sentences.]
```

---

## Rules

- **All four tables, every family, all six months.** No shortcuts — the whole point of this capstone is that S&OP doesn't get to solve one easy family and skip the rest.
- **Every gap-closing action needs a cost attached.** "We used overtime in April" is incomplete; "we used 950 units of April overtime at $31.00/unit, $29,450" is the standard.
- **The executive summary is exactly what it says — one page.** If it doesn't fit, you're including detail that belongs in `balance-tables.sql` instead. Practicing what to leave out is part of the assignment.
- **State your recommendation and defend it.** "Approve" with no reasoning is not a complete summary.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Correctness | 30% | Balance tables and reconciliation are arithmetically right for all 3 families |
| Feasibility | 20% | The recommended plan genuinely never breaches any family's safety-stock target |
| Cost discipline | 15% | Gap-closing actions are costed, and cheaper options are chosen where available |
| Financial reconciliation | 20% | Revenue variance vs. `financial_targets` is computed correctly and explained, not just reported |
| Executive summary | 15% | Fits one page, follows the required structure, ends with a clear, justified recommendation |

---

## Reflection (`notes.md`, ~200 words)

1. Which family required the most judgment, and why — was it a capacity problem, a demand-forecast problem, or a finance-target problem?
2. Where did the operational plan (units) and the financial plan (dollars) tell two different stories about the same month? How did you reconcile them?
3. If you had to recommend Crunch Gear invest in **one thing** before next quarter's cycle (more capacity, better forecasting, more subcontract relationships, something else), what would it be, based on what this cycle showed you?
4. What would break first if demand came in 15% higher than the base case across all three families at once? (Foreshadows Week 11 — risk and resilience.)

---

## Why this matters

This mini-project is the shape of a real monthly S&OP analyst's job: four separate data sources, a capacity constraint that doesn't care what Sales wants, a finance target built on assumptions that are already stale, and one page to tell leadership the truth and recommend a decision. Do it once, completely, end to end, and the whole rest of this course's "optimize the network" work (Weeks 9, 11, 12) has a concrete, numbers-backed plan to optimize *against* instead of an abstract one.

When done: push, then take the [quiz](../quiz.md) and preview [Week 11 — Risk, Resilience & the Digital Supply Chain](../../week-11-risk-resilience-and-digital-supply-chain/).
