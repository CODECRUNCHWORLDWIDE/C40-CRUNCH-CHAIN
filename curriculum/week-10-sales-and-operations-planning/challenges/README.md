# Week 10 — Challenges

Two open-ended problems. Unlike the exercises, these reward judgment and a correctly-verified plan over a single right answer — do them after all three exercises.

1. **[Challenge 1 — Build a Constrained S&OP Plan](challenge-01-constrained-sop-plan.md)** — find a full six-month Trail Footwear production plan that never stocks out, using regular capacity, overtime, and subcontract capacity across the whole horizon, at the lowest added cost you can manage. *(~2h.)*
2. **[Challenge 2 — Scenario-Planning Model](challenge-02-scenario-planning-model.md)** — build base/upside/downside scenarios for Trail Footwear's second half and recommend which one Crunch Gear should actually plan capacity to. *(~90 min.)*

## How these are judged

Neither challenge has a single numeric answer key the way the exercises do. Instead, each one tells you what a *strong* submission looks like. You're being graded on:

- **Feasibility, proven, not asserted.** For Challenge 1, "my plan works" is not a finding — a table showing every month's ending inventory at or above the safety-stock target, with the SQL or Python that produced it, is.
- **Quantified comparison.** "Pre-building in Q1 is cheaper" is not a finding; "pre-building in Q1 costs $302,100 in overtime/subcontract premiums and avoids a $94,250 April revenue shortfall" is.
- **Stated reasoning.** For Challenge 2, every scenario recommendation needs to name the specific trade-off (cost of being wrong on the downside vs. cost of under-investing on the upside), not just a chosen number.

Keep your work in `challenge-01.sql` (or `.sql` + `.py`) and `challenge-02.py`/`.md` with your code **and** your written reasoning. The reasoning is the point.
