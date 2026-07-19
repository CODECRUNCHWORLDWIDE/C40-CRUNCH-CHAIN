# Exercise 1 — Compute EOQ for a Catalog

**Goal:** Turn Lecture 1's EOQ formula into a query and a pandas script you run once against the whole catalog, then use to answer real questions about it. By the end, "compute the EOQ for a SKU" should feel completely mechanical.

**Estimated time:** 1 hour.

## Setup

Confirm the seed table is loaded:

```sql
SELECT COUNT(*) FROM skus;   -- must print 10
```

## Tasks

### Part A — SQL

1. **Holding cost per unit.** Write a query that returns `sku_id`, `sku_name`, and `h` = `unit_cost * holding_pct`, rounded to 2 decimals, for all 10 SKUs.

2. **EOQ.** Extend the query to add `eoq = SQRT(2 * annual_demand * order_cost / h)`, rounded to 1 decimal.

3. **Order frequency and cycle time.** Add `orders_per_year = annual_demand / eoq` and `cycle_days = 365 / orders_per_year`, both rounded to 1 decimal.

4. **Minimum total cost.** Add `tc_at_eoq = SQRT(2 * annual_demand * order_cost * h)`, rounded to 2 decimals. *(This is Lecture 1 section 4's shortcut formula — it should match `(annual_demand/eoq)*order_cost + (eoq/2)*h` to within rounding. Compute both ways for SKU 1 and confirm they agree — that's your correctness check.)*

5. **Rank by total cost.** Which SKU carries the highest `tc_at_eoq`? Which the lowest? Are they the highest/lowest by annual demand too, or does the ranking shuffle? Write one sentence explaining why (or why not) demand alone predicts total inventory cost.

### Part B — pandas

6. Load the `skus` table into a DataFrame and reproduce columns `h`, `eoq`, `orders_per_year`, `cycle_days`, `tc_at_eoq` using vectorized pandas/numpy operations (no `.apply()` with a Python loop — use direct column arithmetic and `np.sqrt`).

7. **Sensitivity check.** For SKU 1 (Alpine Shell Jacket) only, compute `TC(Q)` at `Q = 0.5*EOQ`, `Q = 0.8*EOQ`, `Q = EOQ`, `Q = 1.2*EOQ`, and `Q = 2*EOQ`. Report each as a percentage above the minimum (`TC(Q)/TC(Q*) - 1`). Confirm your numbers roughly match the ratios in Lecture 1 section 6's table (1.7%, 2.5%, 8.3%, 25% — allow some rounding drift since the lecture's ratios used exact multiples of `r`).

8. **Round-number policy.** Real purchase orders round to whole units (you can't order 234.2 jackets). Recompute `tc_at_eoq` for SKU 1 using `Q = round(EOQ)` instead of the exact `EOQ`, and report the dollar difference from the true minimum. Confirm it's small — this demonstrates why rounding your EOQ to the nearest practical order size is safe.

## Expected results (spot checks)

- SKU 1 (Alpine Shell Jacket): `h ≈ 21.00`, `eoq ≈ 234.2`, `orders_per_year ≈ 20.5`, `tc_at_eoq ≈ 4918.5`.
- SKU 8 (Insulated Water Bottle): `h = 3.60`, `eoq ≈ 632.5`.
- SKU 2 (Summit Down Parka) should have one of the **smallest** EOQs in the catalog despite being far from the smallest-demand SKU — low volume *and* high unit cost both push EOQ down.

## Done when…

- [ ] All 10 SKUs have `h`, `eoq`, `orders_per_year`, `cycle_days`, and `tc_at_eoq` computed in both SQL and pandas, and the two agree to rounding.
- [ ] Task 4's two ways of computing `tc_at_eoq` match for SKU 1.
- [ ] Task 7's sensitivity percentages are in the same ballpark as Lecture 1's table.
- [ ] Task 8 shows the rounding penalty is small (well under 1% of `tc_at_eoq`).

## Stretch

- Add a column `annual_purchase_cost = annual_demand * unit_cost` (the cost of the goods themselves, excluded from `TC` on purpose) and a column `total_annual_cost = annual_purchase_cost + tc_at_eoq`. For which SKU is `tc_at_eoq` the largest *fraction* of `total_annual_cost`? What does that tell you about which SKUs are worth the most attention when tuning inventory policy vs. which are dominated by the cost of the goods themselves?

## Submission

Commit `exercise-01.sql` and `exercise-01.py` to your portfolio under `c40-week-05/exercise-01/`.
