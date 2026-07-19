# Challenge 1 — End-to-End Optimization Run

**Time:** ~2.5 hours · **Tools:** Python (pandas, numpy, pulp)

Exercises 1–3 built the pipeline in three separate pieces. This challenge chains them into one script and then breaks it on purpose — a demand shock, a capacity cut — because a pipeline that only ever runs on the one clean month it was built against isn't actually automated, it's just a demo.

## Part A — One script, twelve months (60 min)

Write `run_pipeline.py` implementing `run_capstone_pipeline(engine, target_month)` from Lecture 2 §5, then call it for **all 12 calendar months** (`"2026-01"` through `"2026-12"`), collecting each month's `total_cost`, `outbound_cost`, and `holding_cost` into one results table.

Produce a chart or table (`monthly_costs.csv` + a short plot, matplotlib is fine) showing total in-scope cost by month. Answer: which month has the highest total cost, and does that track with the demand generator's seasonal peak (Lecture 1 §4 — August through November)? State the peak month's total cost and how much higher it is than the lowest-cost month, in dollars and percent.

## Part B — Demand shock (45 min)

Crunch Gear's sales team just forecast a **+20% demand shock** for the Northeast region only, starting next month (a large new wholesale account). Re-run the pipeline for that month with `region_demand["NE"]` scaled by 1.20, holding every other region's demand at its normal forecast.

Report:
1. Does the LP still return `"Optimal"`, or does some DC blow past capacity? If capacity is still respected in aggregate but the *mix* changes, show exactly which lane(s) picked up the extra volume and at what marginal cost per unit versus the pre-shock plan.
2. What is the new total in-scope cost, and how much of the increase is attributable to the shock itself versus a shift to a more expensive lane to absorb it?

## Part C — Capacity cut (45 min)

Memphis DC's landlord informs Crunch Gear that due to a lease renegotiation, Memphis's usable capacity is dropping from 22,000 to **17,000 units/month**, effective next month, with no change to the fixed cost. Re-run the pipeline with this new `CAPACITY["MEM"] = 17000` at normal (non-shocked) demand.

Report:
1. The new optimal plan and total cost.
2. How much more expensive is this capacity-constrained plan than the unconstrained optimum from Exercise 3, in dollars per month and percent?
3. If Crunch Gear could pay to keep Memphis at 22,000 units of capacity, what is the maximum monthly amount they should be willing to pay for that extra 5,000 units of capacity, based purely on the cost difference you just computed? (This is exactly the *shadow price* of the capacity constraint — if you're using PuLP, you can also read this directly off the solved model's constraint duals; look up `constraint.pi` in PuLP's documentation and compare it to your hand-computed answer.)

## What "strong" looks like

- Part A's monthly run is a genuine loop over 12 real forecasts, not 12 copies of the same number.
- Part B correctly distinguishes "still feasible, just costlier" from "infeasible" — and if you engineer a scenario that goes infeasible, you say so and explain why, rather than silently reporting a wrong number.
- Part C's shadow-price answer is grounded in an actual before/after cost comparison (or PuLP's `.pi` attribute), not a guess.
- All three parts run from `run_pipeline.py` with a `--scenario` flag or three separate documented function calls — not three different hand-edited copies of the script.

## Deliverable

`challenge-01/run_pipeline.py`, `challenge-01/monthly_costs.csv` (+ chart), and `challenge-01/results.md` with Parts A–C's findings, each with the specific dollar and percent figures requested above.
