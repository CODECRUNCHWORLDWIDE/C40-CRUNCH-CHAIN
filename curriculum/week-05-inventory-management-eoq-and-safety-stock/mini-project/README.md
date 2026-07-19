# Mini-Project — A Full-Catalog Stocking Policy at 95% Service

> Set an EOQ, reorder point, and safety-stock policy for the entire 10-SKU Austin East catalog, hit a **95% cycle-service level on every SKU**, and report the total annual inventory cost — all computed on SQL data, in pandas. No spreadsheets.

**Estimated time:** 3 hours, best done Saturday after the exercises and challenges.

This is the week's capstone, and it's the actual job. Nobody at Crunch Gear is going to hand you a single SKU and ask for its EOQ in isolation — they're going to hand you a catalog and a budget conversation, and expect a policy table plus a number: "this is what it costs us to run this catalog at 95% service, here's the SKU that's most expensive to protect, and here's what raising the bar to 98% would cost on top."

---

## Deliverable

A directory in your portfolio `c40-week-05/mini-project/` containing:

1. `seed.sql` — the `skus` table from the week README (copy it in, so this project is self-contained).
2. `policy.py` — one script that reads `skus` from your database, computes the full stocking policy, and writes the results back out.
3. `stocking_policy.csv` — the output table (see schema below).
4. `report.md` — the written summary (see structure below).
5. `notes.md` — a short reflection (see the end).

---

## Required computations

For **every one of the 10 SKUs**, compute and include in `stocking_policy.csv`:

| Column | Meaning |
|---|---|
| `sku_id`, `sku_name`, `category` | identifiers |
| `eoq` | Economic Order Quantity (Lecture 1) |
| `orders_per_year` | `annual_demand / eoq` |
| `daily_demand_mean` | `annual_demand / 365` |
| `sigma_dlt` | combined demand-during-lead-time std dev (Lecture 2) |
| `safety_stock` | at a **95%** cycle-service level, `z` = 1.645 |
| `reorder_point` | `daily_demand_mean * lead_time_days + safety_stock` |
| `cycle_stock_holding_cost` | `(eoq/2) * unit_cost * holding_pct` — annual $ cost of the "regular" order-cycle inventory |
| `safety_stock_holding_cost` | `safety_stock * unit_cost * holding_pct` — annual $ cost of the buffer |
| `ordering_cost_annual` | `orders_per_year * order_cost` |
| `total_annual_policy_cost` | sum of the three cost columns above |

That's a full **(s,Q)** policy per SKU — the reorder point `s` from Lecture 2, the order quantity `Q` from Lecture 1 — plus every cost component broken out so you can see exactly where the money goes.

---

## `report.md` — required sections

### 1. Executive summary (≤150 words)

The total annual policy cost across the catalog, broken into the three cost components (cycle stock holding, safety stock holding, ordering), and the one or two SKUs that dominate the total cost — with a one-sentence explanation of *why* those SKUs dominate (high volume? high unit cost? long, variable lead time? some combination?).

### 2. Full policy table

Render `stocking_policy.csv` as a markdown table, sorted by `total_annual_policy_cost` descending.

### 3. Service-level sensitivity

Recompute `safety_stock`, `reorder_point`, and `safety_stock_holding_cost` at **90%** and **99%** targets (in addition to the required 95%), for every SKU. Report:
- The total catalog-wide `safety_stock_holding_cost` at each of the three service levels.
- The dollar jump from 95% → 99%, and what percentage of the 95%-level total policy cost that jump represents.
- A one-paragraph recommendation: is a blanket 99% target across the whole catalog worth it, or would you recommend a **differentiated** service level (e.g., 99% on the highest-margin/highest-visibility SKUs, 90–95% on the rest)? Defend your answer with the numbers you just computed, not a general opinion.

### 4. One SKU, fully worked

Pick the SKU with the **highest `total_annual_policy_cost`**. Show every step of its calculation by hand (not just the script output) — `H`, `EOQ`, `σ_DLT`, `SS`, `ROP`, and every cost line — the way Lecture 1's and Lecture 2's worked examples did. This is the section that proves you understand the mechanics, not just that your script runs.

---

## Milestones

- **Milestone 1 (45 min):** Seed the database, write and test the EOQ portion of `policy.py` against Exercise 1's expected values.
- **Milestone 2 (45 min):** Add the safety-stock and reorder-point portion, matching Exercise 2's expected values.
- **Milestone 3 (45 min):** Add every cost column, produce `stocking_policy.csv`, and sanity-check the totals (do cycle-stock holding cost and ordering cost come out roughly equal per SKU, the way Lecture 1 said they should at the EOQ optimum?).
- **Milestone 4 (45 min):** Service-level sensitivity sweep, `report.md`, and the fully-worked SKU section.

---

## Rules

- **SQL + pandas only.** The `skus` table lives in Postgres or SQLite; `policy.py` reads it with `pd.read_sql` (or `sqlite3` + `pd.read_sql_query`) and does every computation in pandas/numpy. No spreadsheet touches this data at any point — that's the whole point of this course's data rule.
- **One target service level baked into the primary table** (95%), with the 90%/99% sensitivity as clearly separate, labeled output — don't blend them into one ambiguous table.
- **Every dollar figure needs its formula traceable** — a reviewer should be able to look at any cost column and know exactly which lecture's formula produced it.
- **Round for display, not for calculation** — carry full precision through your pandas computation, and only round when you write out the final table/report.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Correctness | 35% | EOQ, safety stock, ROP, and every cost column match the exercise-verified formulas exactly |
| Completeness | 20% | All 10 SKUs, all required columns, all three service levels in the sensitivity section |
| Sensitivity analysis | 20% | 90/95/99% comparison is computed correctly and the recommendation is defended with the actual numbers, not a hand-wave |
| Worked example | 15% | The highest-cost SKU's full by-hand calculation is shown, matching the script's output |
| Communication | 10% | `report.md` reads like something you'd actually send a manager — clear numbers, clear recommendation |

---

## Reflection (`notes.md`, ~200 words)

1. Which cost component — cycle-stock holding, safety-stock holding, or ordering cost — turned out to be the largest share of total catalog cost? Did that surprise you?
2. Which SKU would you push hardest to *reduce* lead-time variability on, and why (tie it back to `σ_DLT`'s two terms from Lecture 2)?
3. If Crunch Gear's finance team offered you a choice — cut supplier lead time by 20%, or cut lead-time variability (`σ_L`) by 20%, for one SKU of your choice — which would you pick, for which SKU, and why? (There's no single right answer; defend your reasoning using the formulas from this week.)
4. What would you need in order to move from a **95% cycle-service level** target to a **95% fill-rate** target instead? Which lecture covered the difference, and why does the choice matter to a stakeholder?

---

## Why this matters

Every real inventory-planning job comes down to exactly this exercise, at a bigger scale: take a catalog, apply consistent formulas, produce a policy and a cost number, and be ready to defend both the mechanics and the trade-off when someone asks "what if we wanted better service?" Do this once carefully on 10 SKUs and the same script — parameterized, checked into version control — is what a real operations team would run against a catalog of ten thousand SKUs next quarter.

When done: push, then take the [quiz](../quiz.md) and start [Week 6 — Procurement, spend analysis, supplier scorecards & lead-time risk](../../week-06-procurement-spend-analysis-and-supplier-risk/).
