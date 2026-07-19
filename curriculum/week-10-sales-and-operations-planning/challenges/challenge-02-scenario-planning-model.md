# Challenge 2 — Scenario-Planning Model

**Time:** ~90 minutes. **Difficulty:** Medium-Hard. **No single right recommendation.**

## The scenario

Recall from Lecture 1: Trail Footwear's April consensus forecast (12,800 units) is a negotiated midpoint between the statistical model (12,200 units, no visibility into the new retail launch) and Sales' number (13,400 units, assuming the launch hits full run rate immediately). That's one month's disagreement. Now Crunch Gear's leadership wants to know: **what does the whole rest of the launch look like if Sales is right, and what does it look like if the statistical model is right** — not just for April, but carried through May and June, since a launch that hits big in April usually keeps building, and a launch that slips usually keeps slipping.

Your job: build three parallel six-month forecasts for Trail Footwear — base, upside, downside — check each one against **maximum available supply** (regular + overtime + subcontract), and recommend whether Crunch Gear should do anything **now**, before the outcome is known, to prepare for the upside case.

## Your task

### 1. Define three scenarios, precisely

Lecture 3 was explicit: a scenario has to be a **named, specific, testable assumption**, not a vague "what if it goes well." Use these three, built from data you already have:

- **Base case:** the `consensus_forecast_units` already in `demand_plan` — no changes.
- **Downside:** "the retail launch slips past this horizon entirely." Use `stat_forecast_units` for April, May, and June (the statistical model's pre-launch baseline) instead of the consensus number; keep January–March unchanged (there's no launch effect there).
- **Upside:** "the launch hits full run rate starting in April, and the momentum carries into May and June." Use `sales_input_units` for April (13,400); for May and June, apply the **same proportional lift** April's `sales_input_units` shows over its `consensus_forecast_units` (`13400 / 12800`) to May's and June's consensus numbers, rounded to the nearest 100 units. Keep January–March unchanged.

Build a small table (SQL temp table, CTE, or a pandas DataFrame — your choice) with columns `month, base_units, downside_units, upside_units`.

### 2. Check each scenario against maximum available supply

For every month, compute `max_supply = regular_capacity_units + overtime_capacity_units + subcontract_capacity_units` from `supply_plan`. For each of the three scenarios, run the same style of cumulative feasibility check from [Challenge 1](./challenge-01-constrained-sop-plan.md): starting from `beginning_inventory_jan`, does the running balance ever go negative, or ever drop below the 900-unit safety-stock target, if you use *maximum* available supply every month?

### 3. Total each scenario over the six months

Sum each scenario's six months of demand and compare the total against the six-month total of `max_supply`. This is the fastest way to see whether a scenario is even *theoretically* satisfiable before checking the month-by-month timing.

### 4. Recommend

Write `challenge-02.md` answering:

1. Which scenario(s), if any, cannot be fully supplied **even at maximum capacity, in any month**? Quote the exact shortfall in units.
2. If the upside scenario is even a realistic possibility (you don't need to estimate its probability precisely — just argue whether it's plausible enough to matter), what could Crunch Gear do **now**, before April, to be ready for it? Name at least one specific lever (e.g., negotiating more subcontract capacity, pre-building extra buffer beyond what Challenge 1's base-case plan already builds, accepting a longer lead time on hiring).
3. What's the cost of preparing for an upside that **doesn't** happen — i.e., what's wasted if Crunch Gear invests in extra capacity/buffer for the upside scenario and the launch actually undershoots to the downside or base case instead? State this trade-off explicitly; don't just recommend "prepare for the upside" without naming what that preparation costs if it turns out to be unnecessary.

## Constraints

- All three scenarios reuse the same `supply_plan` capacity ceilings — you are not allowed to invent extra capacity that doesn't exist in the data for this challenge (Part 4 can *recommend* seeking more, but Parts 1–3 must work within what's given).
- Show your work for the proportional-lift calculation in the upside scenario (Part 1) — state the exact May and June upside numbers you computed and how you rounded them.

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Scenario definitions | Vague ("things go well" / "things go badly") | Specific, numeric, traceable to a stated assumption (per Lecture 3 §5) |
| Feasibility check | Eyeballed | Shown with an actual month-by-month or cumulative-total query/table |
| Trade-off reasoning | Recommends preparing for upside with no cost attached | States the cost of over-preparing *and* the cost of under-preparing, and weighs them explicitly |
| Numbers | Approximate or unverified | Upside/downside totals and the max-supply comparison are computed, not estimated |

## Submission

Commit your scenario table, feasibility checks, and `challenge-02.md` to your portfolio under `c40-week-10/challenge-02/`.
