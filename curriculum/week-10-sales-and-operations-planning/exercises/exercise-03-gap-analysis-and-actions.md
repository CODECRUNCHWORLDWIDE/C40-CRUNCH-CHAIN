# Exercise 3 — Gap Analysis & Actions

**Goal:** find exactly which family/month breaches the safety-stock floor even after bringing overtime and subcontract capacity into play, quantify the leftover gap in both units and dollars, and write a four-part gap-closing recommendation in the shape Lecture 3 taught.

**Estimated time:** 60 minutes.

## Setup

You'll build on Exercise 1's balance table (regular capacity only) and bring in `overtime_capacity_units`, `subcontract_capacity_units`, and their costs from `supply_plan`, plus `unit_price` from `product_reference` and `revenue_target` from `financial_targets`.

## Tasks

1. **Confirm the baseline problem.** Rerun (or reuse) Exercise 1's regular-only balance table for Trail Footwear. Confirm March is `BELOW TARGET` (ending inventory 800 vs. a 900 target) and April–June are all `STOCKOUT`.

2. **Close March with the minimum needed.** March's shortfall against the 900 target is exactly 100 units. Trail Footwear's March `overtime_capacity_units` is 950 at `unit_cost_overtime = 31.00`. Compute the cost of closing March's gap with the *minimum* overtime needed (not the maximum available). *(Expected: 100 units × $31.00 = $3,100. New March ending inventory: exactly 900.)*

3. **Check April with its own maximum levers.** Starting from March's now-fixed ending inventory of 900, compute April's ending inventory if you use April's **maximum** overtime (950 units) **and** maximum subcontract (800 units) on top of April's regular capacity (9,500 units) against April's demand (12,800 units).

   ```sql
   SELECT s.regular_capacity_units + s.overtime_capacity_units + s.subcontract_capacity_units AS april_max_supply,
          d.consensus_forecast_units,
          900 + (s.regular_capacity_units + s.overtime_capacity_units + s.subcontract_capacity_units)
              - d.consensus_forecast_units AS april_ending_inventory_if_maxed
   FROM supply_plan s
   JOIN demand_plan d USING (month, product_family)
   WHERE s.product_family = 'Trail Footwear' AND s.month = '2025-04-01';
   ```

   *Expected: April's maximum available supply is 11,250 units against 12,800 units of demand — even fully maxed, and even starting from a healthy 900-unit March ending balance, **April still ends the month with a shortfall.** Write down the exact number your query returns.*

4. **Quantify the leftover gap in dollars.** Using Trail Footwear's `unit_price` from `product_reference`, convert the unit shortfall from Task 3 into a dollar figure — this is the revenue at risk if nothing further is done before April.

5. **Confirm the other two families need no action.** Run the same regular-capacity-only balance table (Exercise 1's Task 3 query, filtered to each family) for **Backpacks & Bags** and **Apparel**. Confirm both stay `OK` every month with regular capacity alone — no overtime or subcontract required, and no further action needed for either family this cycle.

6. **Write the recommendation.** In `gap-memo.md`, write Trail Footwear's April gap-closing recommendation using Lecture 3 Section 6's four-part shape:
   1. **The number** — the exact leftover shortfall from Task 3, and its dollar value from Task 4.
   2. **The cause** — why April specifically, in one sentence (tie it to the capacity ceiling, not to weak demand).
   3. **At least two costed options** — e.g., pre-build extra buffer in January/February using their currently-unused overtime capacity (cost this out — both months have 850 units of unused overtime at $31.00/unit), vs. accepting the leftover shortfall as a planned, communicated backorder.
   4. **A recommendation**, stated in one sentence, with your reasoning.

## Expected result (spot checks)

- Task 2: March's gap closes for **$3,100**, ending exactly at the 900-unit target.
- Task 3: April's maximum available supply is **11,250 units**; starting from March's fixed 900-unit balance, April still ends at **900 + 11,250 − 12,800 = −650** — a **650-unit stockout**, even after using every lever April itself has.
- Task 4: 650 units × $145.00 = **$94,250** of revenue at risk in April alone if nothing more is done.
- Task 5: Backpacks & Bags and Apparel both stay `OK` all six months on regular capacity alone — confirm this rather than assume it.

## Done when…

- [ ] Tasks 1–5 are answered with queries in `solutions.sql`, each under a `-- Task N` comment.
- [ ] Your Task 3 result matches the −650 spot check.
- [ ] `gap-memo.md` has all four parts, in order, and Part 3 has at least two *costed* options (a dollar figure attached to each, not just a description).
- [ ] You can explain in one sentence why fixing March **first** (Task 2) still isn't enough to save April — the shortfall is bigger than any single month's own capacity can close.

## Stretch

- April's 650-unit leftover gap is smaller than what a *fully* maximized January–March would have prevented (see Lecture 1 and [Challenge 1](../challenges/challenge-01-constrained-sop-plan.md) for the full picture). Compute how many extra units January and February alone would need to produce, beyond meeting their own demand, to fully absorb April's 650-unit gap using their unused overtime capacity. Is 650 units achievable from Jan+Feb overtime alone (850 + 850 = 1,700 units available)? At what added cost?

## Submission

Commit `solutions.sql` and `gap-memo.md` to your portfolio under `c40-week-10/exercise-03/`.
