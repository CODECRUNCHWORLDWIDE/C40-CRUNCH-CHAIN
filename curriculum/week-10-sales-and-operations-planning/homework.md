# Week 10 — Homework

Five problems, ~5 hours total, spread across the week. A mix of hands-on SQL, a model-extension exercise, and two written-reasoning tasks. Commit each.

All SQL runs against `demand_plan`, `supply_plan`, `product_reference`, and `financial_targets` from the [README](./README.md) unless a problem says otherwise.

---

## Problem 1 — Twenty warm-up queries (75 min)

Write and run each. Put them in `warmups.sql` with a `-- N` comment and the answer beneath each.

1. Total `consensus_forecast_units`, summed across all six months, per `product_family`.
2. Which `product_family` has the highest total `revenue_target` across the whole horizon?
3. Every family/month where `sales_input_units - stat_forecast_units` exceeds 500 units.
4. The single family/month with the largest gap between `sales_input_units` and `stat_forecast_units`, anywhere in the table.
5. Average `unit_cost_overtime`, per `product_family`, across all six months.
6. Every family/month where `subcontract_capacity_units = 0`.
7. Total overtime capacity available (`SUM(overtime_capacity_units)`), per `product_family`, across the whole horizon.
8. The `product_family` with the lowest `gross_margin_target_pct`.
9. For every family, compute `standard_margin_pct = 100.0 * (unit_price - standard_unit_cost) / unit_price` from `product_reference`.
10. Every family/month where `regular_capacity_units < consensus_forecast_units` (the raw, single-month shortfall — no running balance, just this month in isolation).
11. Total `revenue_target` across all three families, **January only**.
12. Which single month has the highest **combined** `consensus_forecast_units` summed across all three families?
13. Every `supply_plan` row where `unit_cost_subcontract > unit_cost_overtime * 1.1` (subcontract meaningfully pricier than overtime that month).
14. For each family, the ratio of April's `consensus_forecast_units` to January's (a rough seasonal ramp factor). Which family ramps the hardest?
15. Total `beginning_inventory_jan`, summed across all three families.
16. Every family/month where `consensus_forecast_units = sales_input_units` **exactly** — i.e., no negotiation gap at all that month.
17. The single cheapest `unit_cost_regular` anywhere in `supply_plan`, and which family/month it belongs to.
18. Trail Footwear's total maximum available supply (`regular + overtime + subcontract`) for **June only**.
19. For every family, is `gross_margin_target_pct` above or below its own `standard_margin_pct` (from Task 9)? What does it mean, in plain English, when the *target* is below the *standard* margin?
20. `SELECT COUNT(DISTINCT month) FROM demand_plan;` — should be 6. If it isn't, something's wrong with your seed data; fix it before continuing.

---

## Problem 2 — Extend the model with a fourth product family (75 min)

Crunch Gear is launching a new **"Camp & Outdoor Cooking"** family in July, and wants next quarter's S&OP inputs ready ahead of time.

1. Write `INSERT` statements adding **two new months (July, August 2025)** for this new family to all four tables: `demand_plan`, `supply_plan`, and one new row in `product_reference` (pick your own reasonable `unit_price`, `standard_unit_cost`, `beginning_inventory_jan` — reinterpret as "beginning inventory at family launch" — `safety_stock_target`, `holding_cost_per_unit_month`, and `gross_margin_target_pct`), and `financial_targets`.
2. Invent demand and capacity numbers that are internally consistent with the story of a **brand-new product launch**: low beginning inventory, capacity ramping up (not flat) across the two months, and demand you'd expect to grow as retail awareness builds.
3. Run the Exercise 1-style balance table against your new family for its two months. Does your invented plan clear its own safety-stock target, or did you accidentally create a shortage?
4. **Deliver** `new-family.sql` with the inserts and the balance-table check, plus 3-4 sentences explaining the numbers you chose and why they're realistic for a launch month.

---

## Problem 3 — Why "cost per unit" isn't enough during a gap (30 min)

In `cost-writeup.md`, answer in prose (no more than 350 words total):

1. Trail Footwear's regular unit cost is $22.00. Its April overtime costs $31.00/unit and subcontract $34.00/unit — 41% and 55% premiums respectively. Explain, in your own words, why evaluating "is it worth closing April's gap" using *only* the $22.00 regular cost (instead of the actual $31.00/$34.00 cost of the units that would close it) would lead to the wrong decision.
2. In Exercise 3, you found April still runs a 650-unit stockout after using its own maximum overtime and subcontract, worth $94,250 in at-risk revenue. Under what circumstance would it actually be *cheaper* for Crunch Gear to accept that 650-unit stockout rather than pay to prevent it? Name a real cost that would need to be quantified to make that call properly (that this week's tables don't give you).
3. If you were advising Crunch Gear's CFO, what's one thing you'd ask Finance to change about how `financial_targets.revenue_target` gets built, based on what you saw reconciling it against the consensus forecast this week?

---

## Problem 4 — Three-family scorecard, one query (45 min)

In `scorecard.sql`, write a **single query** (one `SELECT`, CTEs allowed) that returns, for every `product_family`: total `consensus_forecast_units` across the horizon, total achievable units using **regular capacity only**, the **gap** between them, and a `status` column (`'FEASIBLE ON REGULAR'` if the gap is `>= 0`, else `'NEEDS OVERTIME/SUBCONTRACT'`).

**Deliver** the query plus 2-3 sentences: which family needs the most help, by how much, and does that match what you already found in Exercise 1 and Exercise 3?

---

## Problem 5 — A demand "what if" (60 min)

Two weeks before April, word comes back that the retail-partner launch behind Trail Footwear's demand bump has been **pushed to Q3** — the exact "downside" scenario from [Challenge 2](./challenges/challenge-02-scenario-planning-model.md), if you built it.

1. Revert April, May, and June's Trail Footwear demand to `stat_forecast_units` (12,200 / 13,800 / 12,800) instead of `consensus_forecast_units`.
2. Rerun the regular-capacity-only balance table with this new, lower demand. Does Trail Footwear still breach its safety-stock target anywhere?
3. If you built [Challenge 1](./challenges/challenge-01-constrained-sop-plan.md)'s gap-closing plan (extra overtime/subcontract already scheduled to cover the *original*, higher demand), how many units of that planned overtime/subcontract are now **unnecessary**, and what's the dollar cost of not cancelling it in time?

**Deliver** `demand-revision.sql` with both balance tables (original vs. revised demand) side by side, plus 3-4 sentences on what this tells you about the cost of reacting to a demand-planning change *late* versus catching it at the next monthly cycle.

---

## Time budget

| Problem | Time |
|--------:|----:|
| 1 | 75 min |
| 2 | 75 min |
| 3 | 30 min |
| 4 | 45 min |
| 5 | 60 min |
| **Total** | **~4.75 h** |

After homework, take the [quiz](./quiz.md) and ship the [mini-project](./mini-project/README.md).
