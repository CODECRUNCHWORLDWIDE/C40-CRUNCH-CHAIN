# Mini-Project — Plant-to-DC Network Flow, Solved and Stored in SQL

> Build a complete pipeline: read Crunch Gear's plant capacities, DC demand, and lane costs **from SQL**, solve the minimum-cost transportation problem in **PuLP**, and write the optimal shipping plan and its shadow prices **back into SQL** — the real shape of this work, end to end, no notebook full of hard-coded numbers left behind.

**Estimated time:** 3 hours, best done Saturday after the exercises and challenges.

This is the week's capstone, and it's deliberately closer to what an actual network-planning analyst delivers than anything else this week: not just an optimal answer printed to a terminal, but a repeatable pipeline — data in, solve, data out — that a colleague (or a scheduled job) could re-run next month when the demand forecast changes, without touching your code.

---

## Deliverable

A directory in your portfolio `c40-week-09/mini-project/` containing:

1. `solve_network.py` — the full pipeline: connect to SQL, load `plants`/`distribution_centers`/`lane_costs`, build and solve the model, write results back to two new tables.
2. `report.md` — the solved shipping plan in plain English, the total cost, the shadow-price interpretation, and the results of your scenario analysis (Part 4 below).
3. `schema.sql` — the `CREATE TABLE` statements for your two output tables (below), separate from the input schema in the week README so a reviewer can see exactly what you added.

Everything runs against the Week 9 seed network from the [week README](../README.md). Works on PostgreSQL or SQLite; note which you used in `report.md`.

---

## Part 1 — Read the network from SQL

Connect to your `crunch_network` database (or `crunch_network.db` for SQLite) and load the three seed tables into pandas DataFrames, exactly as in Exercise 2. Do not hard-code the plant list, DC list, or costs anywhere in your script — if someone adds a fourth plant to the `plants` table, your script should handle it with zero code changes.

## Part 2 — Solve the transportation problem

Build the model from Lecture 2: one `x[plant, dc]` variable per lane, `≤` supply constraints, `≥` demand constraints, objective = total shipping cost. Confirm `LpStatus == "Optimal"` before proceeding — a script that writes garbage to a database because it never checked solver status is a real, embarrassing bug class, and this project explicitly tests that you guard against it.

## Part 3 — Write results back to SQL

Create two new tables and populate them from your solved model.

```sql
CREATE TABLE optimal_flows (
    plant_id      TEXT    NOT NULL,
    dc_id         TEXT    NOT NULL,
    units_shipped NUMERIC NOT NULL,
    cost_per_unit NUMERIC NOT NULL,
    lane_total_cost NUMERIC NOT NULL,
    PRIMARY KEY (plant_id, dc_id)
);

CREATE TABLE shadow_prices (
    constraint_name TEXT    PRIMARY KEY,
    constraint_type TEXT    NOT NULL,   -- 'supply' or 'demand'
    node_id         TEXT    NOT NULL,
    shadow_price    NUMERIC NOT NULL,
    slack           NUMERIC NOT NULL
);
```

Only insert `optimal_flows` rows where `units_shipped > 0` — a network with a 9-lane cost table but only 5 active lanes should produce a 5-row table, not 9 rows padded with zeros. Populate `shadow_prices` with one row per constraint (supply and demand both), reading `.pi` and `.slack` off each solved `LpConstraint`.

Verify your write with a query, not just a `print` statement:

```sql
SELECT SUM(lane_total_cost) FROM optimal_flows;   -- should equal your solver's total cost
```

## Part 4 — Scenario analysis: what if Ho Chi Minh loses 1,000 units of capacity?

A real network planner never solves a model once and walks away — they ask "what if." Suppose a supplier disruption cuts Ho Chi Minh's monthly capacity from 5,000 to 4,000 units. The Week 9 network is **exactly balanced** (15,000 supply = 15,000 demand), so this isn't a small perturbation — cutting any plant's capacity at all, with demand held fixed, makes the model **infeasible** as originally written. That's not a bug in the exercise; it's the point.

1. First, from Exercise 2, you should already have Ho Chi Minh's shadow price at the *original* capacity: **zero**, despite its supply constraint being fully binding. Write one sentence predicting what you think a 1,000-unit cut will do, *before* you touch the model.
2. Apply the cut (`plants` table edit or a parameterized override — either is fine, say which) and try to re-solve exactly as before. Confirm `LpStatus` reports **infeasible** (or your solver's equivalent) — this is the expected, correct outcome, not an error to debug away.
3. Add a **dummy emergency source** following Lecture 2 Section 6: capacity 1,000 (exactly the shortfall), with a high per-unit cost on every lane out of it (an expedited air-freight buy is a reasonable story — pick a number well above every real lane cost, e.g. $50/unit, and say so). Re-solve.
4. Report: how much of the 1,000-unit shortfall the solver actually routes through the emergency source versus absorbs by reshuffling the real lanes, the new total cost, and the **delta** versus the original $74,200 baseline.
5. Reconcile the result against your Step 1 prediction and Ho Chi Minh's zero shadow price. Explain, in your own words, why a **zero** shadow price on a binding constraint told you nothing reliable about the cost of *cutting* that same resource — a shadow price is a local, small-increase statement about the current optimal corner, not a guarantee about what happens once a cut is large enough to break feasibility entirely.

## Milestones

- **Milestone 1 (45 min):** Data pipeline reading all three seed tables into clean DataFrames, with a sanity-check assertion that supply totals demand.
- **Milestone 2 (45 min):** Model built and solved, `LpStatus` verified, shipping plan printed and matching Lecture 2/Exercise 2's known answer ($74,200).
- **Milestone 3 (45 min):** Both output tables created and correctly populated; the `SUM(lane_total_cost)` verification query passes.
- **Milestone 4 (45 min):** Scenario analysis run, delta reported, and the shadow-price prediction reconciled against the actual re-solve.

## Rules

- **No hard-coded plant/DC/cost data anywhere in `solve_network.py`.** Everything comes from a `SELECT`.
- **Guard the solver status.** Writing to the output tables must be conditional on `LpStatus[prob.status] == "Optimal"` — if a future capacity edit makes the network infeasible, your script should say so clearly and write nothing, not silently insert `None`s.
- **Only positive flows are stored.** Don't pad `optimal_flows` with zero-quantity lanes.
- **Every judgment call gets one sentence in `report.md`** — how you parameterized the scenario override, which SQL engine you used, anything you had to decide that the brief left open.

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Correctness | 30% | Optimal shipping plan and $74,200 baseline total match the known answer |
| SQL round-trip | 25% | Data genuinely read from tables, results genuinely written and independently verifiable by a `SELECT` |
| Shadow-price handling | 20% | Shadow prices captured, stored, and correctly interpreted in the scenario analysis |
| Scenario analysis | 15% | Ho Chi Minh capacity-cut re-solve run correctly, delta computed, reconciled against the shadow price with real reasoning |
| Code quality | 10% | Solver-status guard present; no hard-coded network data; readable variable names |

## Reflection (`notes.md`, ~200 words)

1. What broke first when you tried to make the pipeline fully data-driven (no hard-coded dicts)?
2. Did the shadow price correctly predict the cost of Ho Chi Minh's capacity cut, or did the prediction break down? Why?
3. If Crunch Gear added a fourth plant next quarter, what — precisely — would you have to change in your script? (The honest answer should be "nothing, just insert three rows.")
4. What's one thing about this pipeline that still wouldn't survive contact with a real, 50-plant, 200-DC network? (Foreshadows why later weeks and later courses reach for commercial solvers and warehouse-scale data engineering.)

---

## Why this matters

This is the actual job. Someone hands a network analyst a spreadsheet's worth of capacities and costs — except it shouldn't be a spreadsheet, it should be rows in a database — and the deliverable isn't a clever one-off script, it's a pipeline that still works when next month's numbers change. Read from SQL, solve with a real optimizer, write back to SQL, and you've just built the same shape of tool that runs real supply chains, just smaller.

When done: push, then take the [quiz](../quiz.md).
