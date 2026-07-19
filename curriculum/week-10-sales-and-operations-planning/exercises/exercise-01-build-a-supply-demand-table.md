# Exercise 1 — Build a Supply-Demand Balance Table

**Goal:** turn the four raw seed tables into one running, month-by-month balance table per product family — the core artifact of every S&OP cycle — using a SQL window function to carry inventory forward correctly.

**Estimated time:** 90 minutes.

## Setup

Confirm your four tables are seeded (see the [week README](../README.md)):

```sql
SELECT COUNT(*) FROM demand_plan;      -- 18
SELECT COUNT(*) FROM supply_plan;      -- 18
SELECT COUNT(*) FROM product_reference; -- 3
```

Create a file `solutions.sql` and put each answer under a `-- Task N` comment.

## Background: why this needs a window function, not just a join

A plain join of `demand_plan` and `supply_plan` gives you, for any single month, `regular_capacity_units - consensus_forecast_units` — a **month gap**. But that's not the same as knowing whether you'll actually run out of stock, because a deficit in one month can be absorbed by surplus **carried forward** from an earlier month. You need a **running total**: `ending_inventory[this month] = ending_inventory[last month] + supply[this month] - demand[this month]`, chained all the way back to the known starting point (`beginning_inventory_jan`). That's exactly what `SUM() OVER (PARTITION BY product_family ORDER BY month)` computes.

## Tasks

**This exercise uses `regular_capacity_units` only** — no overtime, no subcontract. Those are levers held in reserve; Exercise 3 brings them in.

1. **The raw month gap.** For every family and month, select `month`, `product_family`, `consensus_forecast_units`, `regular_capacity_units`, and `month_gap = regular_capacity_units - consensus_forecast_units`. *(Expected: 18 rows. Trail Footwear April should show `month_gap = -3300`.)*

2. **Monthly net change, including beginning inventory.** Build a CTE (or subquery) `net_change` per row: `regular_capacity_units - consensus_forecast_units`, same as Task 1 but you'll build on it next.

3. **The running balance — the core task.** Using a window function, compute `ending_inventory` for every family/month as `beginning_inventory_jan + running_sum(net_change)`, where the running sum is `SUM(net_change) OVER (PARTITION BY product_family ORDER BY month)`. Join `product_reference` for `beginning_inventory_jan` and `safety_stock_target`. Order the output by `product_family, month`.

   *Hint — the shape:*
   ```sql
   WITH monthly AS (
       SELECT d.month, d.product_family,
              d.consensus_forecast_units,
              s.regular_capacity_units,
              s.regular_capacity_units - d.consensus_forecast_units AS net_change
       FROM demand_plan d
       JOIN supply_plan s USING (month, product_family)
   )
   SELECT m.month, m.product_family, m.consensus_forecast_units, m.regular_capacity_units,
          r.beginning_inventory_jan
              + SUM(m.net_change) OVER (PARTITION BY m.product_family ORDER BY m.month) AS ending_inventory,
          r.safety_stock_target
   FROM monthly m
   JOIN product_reference r USING (product_family)
   ORDER BY m.product_family, m.month;
   ```

4. **Flag the breaches.** Extend Task 3 with a `status` column: `'STOCKOUT'` if `ending_inventory < 0`, `'BELOW TARGET'` if `0 <= ending_inventory < safety_stock_target`, else `'OK'`.

5. **Summarize by family.** From your Task 4 result, write a query (or a second query against a saved view/CTE) that returns, per `product_family`, the count of months in each status and the single worst (lowest) `ending_inventory` value reached.

## Expected result (spot checks)

Using regular capacity only, the ending-inventory sequence for **Trail Footwear** should be:

```
Jan: 1800   Feb: 1700   Mar: 800   Apr: -2500   May: -5700   Jun: -7700
```

March is the first flag (`800 < 900` target → `BELOW TARGET`), and April through June are all `STOCKOUT`. **Backpacks & Bags** and **Apparel** should both come back entirely `OK` for all six months — regular capacity alone is sufficient for those two families this horizon; only Trail Footwear has a structural problem worth escalating.

## Done when…

- [ ] `solutions.sql` has all 5 tasks, each under a `-- Task N` comment.
- [ ] Task 3's window function is `PARTITION BY product_family` — if you get one long running total that ignores family boundaries, this is the bug to look for first.
- [ ] Your Trail Footwear sequence matches the spot check above exactly.
- [ ] Task 5's summary correctly shows Trail Footwear with 1 `BELOW TARGET` month and 3 `STOCKOUT` months, and both other families with 0 of either.
- [ ] You can explain, in one sentence, why April's `ending_inventory` is more negative than April's raw `month_gap` alone would suggest. *(It isn't — but May and June get progressively worse specifically because the shortfall compounds instead of resetting each month. Make sure you can explain why a running total does that and a plain per-month gap wouldn't show it.)*

## Stretch

- Rerun Task 3 using **cumulative demand vs. cumulative supply** instead of a running net — i.e., two separate running sums (`SUM(consensus_forecast_units) OVER (...)` and `beginning_inventory_jan + SUM(regular_capacity_units) OVER (...)`), then subtract. Confirm you get the identical `ending_inventory` values as the single-running-sum version in Task 3. This is a useful sanity check: two different SQL shapes computing the same underlying quantity should always agree.
- Add a `LAG()` window function to show each month's `ending_inventory` next to the *previous* month's, and compute the month-over-month change directly — a common way this table gets presented in an actual S&OP deck.

## Submission

Commit `solutions.sql` to your portfolio under `c40-week-10/exercise-01/`.
