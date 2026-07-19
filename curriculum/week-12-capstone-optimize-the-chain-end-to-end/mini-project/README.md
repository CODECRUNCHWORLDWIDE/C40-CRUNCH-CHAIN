# Mini-Project — Design, Forecast, Optimize, and Automate a Full Supply Chain

> This is the capstone of **C40 · Crunch Chain**. One deliverable, four parts, one pipeline: assemble a real multi-echelon network in SQL, forecast its demand, size its inventory policy, optimize its flow with linear programming, and automate the whole loop into a single reproducible script — then present the result as an executive recommendation. Every technique from all twelve weeks of this course shows up here, chained into one working system.

**Estimated time:** 6 hours, spread across Thursday–Saturday per the weekly schedule.

## The brief

You are the operations analyst assigned to Crunch Gear's Alpine jacket line. Leadership has asked for one thing: **cut total network cost without hurting customers.** You have the network from this week's lectures and exercises — two plants, three DCs, four regions, six SKUs, 24 months of demand history — and you have twelve weeks of technique behind you. Build the thing a real team would actually ship, then defend it in a room.

This mini-project has two acceptable shapes. **Pick one before you start** and say which one you picked in your `report.md`:

- **Shape A — Deepen this week's network.** Extend the Exercise 1–3 pipeline with real rigor: proper backtesting across multiple forecast methods (not just seasonal-naive), a full 12-month rolling optimization (not one snapshot month), and at least one of this week's Challenge 1 stress tests folded into the automated pipeline itself (not a one-off script).
- **Shape B — Build your own network.** Design a different multi-echelon network of comparable size (2+ supply sources, 3+ DCs or warehouses, 3+ demand regions, 5+ SKUs) for a business you invent or a real company you research, and run the full pipeline against it from scratch. This is more work but more yours — recommended if you want a genuinely novel portfolio piece distinct from every other student's submission.

Whichever shape you pick, the four parts below are non-negotiable.

---

## Part 1 — Assemble the network (SQL)

Build the complete schema: supply sources, distribution nodes, demand regions, SKUs, lane costs and lead times, DC capacities and fixed costs, and at least 12 months of demand history (24 preferred, matching this week's pattern). If you picked Shape A, this is Exercise 1's schema, verified. If you picked Shape B, design and seed your own — reuse Lecture 1 §3–4's table shapes and demand-generator pattern, with your own entities and numbers.

**Required:** every table, every row count, verified with `SELECT COUNT(*)` checks documented in your report — exactly Exercise 1's discipline, not skipped because it's "just setup."

## Part 2 — Forecast + inventory policy (Python)

For every SKU-region (or SKU-node, if your network's demand isn't region-shaped) pair: forecast demand for a defined planning horizon, **backtest the forecast** and report its error (MAE and MAPE, minimum), then compute EOQ, safety stock (at a stated target service level — 95% unless you have a specific, justified reason to pick differently), and reorder point.

**Required:** state your target service level explicitly and justify it in one sentence (this week's default of 95% is a defensible choice, but say why you're using it, not just that you are).

## Part 3 — Network optimization (PuLP or SciPy)

Formulate and solve a linear program that minimizes total network cost — at minimum, outbound freight — subject to demand-satisfaction and capacity constraints. If you're comfortable with the added complexity, extend it to a true multi-echelon flow (plant-to-DC *and* DC-to-region simultaneously, not DC-to-region alone) — this is a natural stretch goal (see below), not a requirement.

**Required:** the solved model's status must be reported (`"Optimal"`, or a documented reason it isn't), and the optimized plan must be stated in a form a warehouse manager could act on (which node ships how much to which destination).

## Part 4 — Automate + present

Chain Parts 1–3 into **one script** — `run_capstone.py` or equivalent — that takes a target planning period as input and produces the full report as output, with no manual steps in between. Then write the executive one-pager per Lecture 3's five-part structure.

**Required:** running your script a second time, on a different target month or a lightly perturbed demand input, must produce a different (correct) result without any code changes — this is the actual test of "automated," not "I wrote functions."

---

## Deliverable

A directory in your portfolio `c40-week-12/mini-project/` containing:

1. `network-schema.sql` — Part 1: the complete schema and seed data.
2. `run_capstone.py` — Part 4: the single script chaining forecast → inventory policy → network optimization → report generation.
3. `pipeline_functions.py` (or inline in `run_capstone.py` if you prefer one file) — Parts 2–3: `forecast_demand()`, `compute_inventory_policy()`, `build_and_solve_network_lp()`, clearly separated and independently testable.
4. `verification.md` — every row-count and sanity check from Part 1, and your backtested forecast error from Part 2.
5. `report.md` — the full analysis: baseline cost and service profile, optimized cost and service profile, the specific levers responsible for the gap between them (state each lever's dollar contribution, per Lecture 2 §6's format), and which scenarios you additionally tested (a demand shock, a capacity change, or both — required, at least one, per Challenge 1's pattern).
6. `memo.md` — the one-page executive recommendation, per Lecture 3's five-part structure.

## Rules

- **SQL and Python only** — no spreadsheet anywhere in the pipeline, per this course's data rule, stated one final time because this is the deliverable someone might actually screenshot into a portfolio.
- The optimization must be a genuine LP solved with PuLP or SciPy — a manually-sorted "assign each region to its cheapest lane" heuristic is not sufficient unless you can also prove (as Lecture 1 §5 and Lecture 2 §4 did, by hand) that it happens to equal the true LP optimum for your specific network. If your network has any binding capacity constraint, the LP requirement is not optional — hand-sorting will get it wrong.
- Every number in `report.md` and `memo.md` must trace to `run_capstone.py`'s actual output. If a number in your memo doesn't match a number your script printed, that's treated as a correctness bug, not a rounding difference.
- The backtest is mandatory even if you're confident the forecast is good — this week's grading specifically checks for a reported MAPE, not just a forecast.

## Stretch goals (optional, not required for a complete submission)

Pick zero, one, or several — these are for students who finish the required parts with time to spare and want to push further:

- **True multi-echelon LP.** Extend the network LP to simultaneously optimize plant→DC *and* DC→region flow (two linked sets of decision variables, with DC throughput as both an inbound and outbound constraint) instead of holding plant sourcing fixed as this week's lectures did.
- **Multiple forecast methods, formally compared.** Run seasonal-naive, moving average, and (per Week 4) an ML-based forecast side by side, backtest all three, and let the pipeline automatically select the lowest-error method per SKU-region rather than assuming one method fits every series.
- **Mixed-integer extension (facility decision).** Add a binary "is this DC open" decision variable and a scenario where opening or closing a DC is itself a lever, not just how much flow it carries — this turns the LP into a MIP and requires `pulp.LpVariable(..., cat="Binary")`.
- **Monte Carlo risk layer.** Instead of a single point-forecast scenario, run the pipeline across 500+ simulated demand draws (sampling from each SKU-region's estimated demand distribution) and report the *distribution* of optimized costs and the probability any DC breaches capacity — a direct extension of Week 11's risk material.
- **A second real scenario, compared.** Run your pipeline against two genuinely different target periods (e.g., a normal month and the seasonal peak) and report how much the *optimal plan itself* changes, not just the cost — this is the strongest possible evidence that your pipeline is actually re-runnable, not a one-off.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Part 1 — data model | 15% | Complete, verified schema; every sanity check documented and passing |
| Part 2 — forecast + policy | 20% | Backtested forecast with a stated, reasonable MAPE; correct EOQ/safety stock/ROP formulas, correctly chained from the forecast's error, not its mean alone |
| Part 3 — network optimization | 25% | A genuine, correctly formulated and solved LP; status checked; plan is actionable and its cost is verified against at least one hand-computable sanity check |
| Part 4 — automation | 15% | One script, no manual steps, demonstrably re-runnable on a different input with a different correct result |
| Analysis quality (`report.md`) | 10% | Baseline vs. optimized cost broken into named levers with dollar amounts; at least one stress scenario tested and reported |
| Executive memo (`memo.md`) | 15% | Follows the five-part structure; headline is one clear sentence with one number; trade-off is real and stated plainly; fits on one page |

## Why this matters

This is the deliverable a hiring manager or a real team lead actually wants to see: not a homework assignment, but a small, complete, working system — data model, statistical forecast, optimization, automation, and a communication artifact a non-technical executive could act on — built end to end by one person, with every number traceable back to code. Every one of the twelve weeks behind you fed into one piece of this. Keep the whole directory; it's the strongest single artifact this course produces, and it's the one most worth putting in a portfolio exactly as it stands.

When done: push, take the [quiz](../quiz.md), and you're finished with **C40 · Crunch Chain**.
