# Exercise 1 — Assemble the Capstone Data Model

**Time:** ~1.5 hours · **Tools:** SQL + Python (pandas, numpy, sqlalchemy)

Build the full network schema from Lecture 1, load it, generate the 24-month demand history, and prove all of it is correct before you build anything on top of it. This exercise produces no analysis — it produces a **trustworthy foundation**, which is the least glamorous and most important task in any capstone.

## Task 1 — Create the schema (30 min)

In a fresh database `crunch_chain_capstone`, run Lecture 1 §3's full schema and seed script: `plants`, `dcs`, `regions`, `skus`, `unit_costs`, `plant_dc_lanes`, `dc_region_lanes`. Do not modify the numbers — every later file in this week (including the quiz answer key) assumes these exact values.

Save the script as `schema.sql`.

## Task 2 — Generate and load demand history (30 min)

Run Lecture 1 §4's Python demand generator **exactly as written**, including `np.random.seed(40)` and the corrected `SEASONAL_INDEX` table (12 values summing to ~12.0, so the annual mean recovers each region's stated base demand). Load the result into a `demand_history` table.

Save the script as `generate_demand.py`.

## Task 3 — Verify everything (30 min)

Run every check below and record the actual output in a `verification.md` file, alongside the expected value. If any check fails, find the bug before moving to Exercise 2 — do not "fix it downstream."

```sql
SELECT COUNT(*) FROM plants;              -- 2
SELECT COUNT(*) FROM dcs;                 -- 3
SELECT COUNT(*) FROM regions;             -- 4
SELECT COUNT(*) FROM skus;                -- 6
SELECT COUNT(*) FROM unit_costs;          -- 12  (2 plants × 6 SKUs)
SELECT COUNT(*) FROM plant_dc_lanes;      -- 6   (2 plants × 3 DCs)
SELECT COUNT(*) FROM dc_region_lanes;     -- 12  (3 DCs × 4 regions)
SELECT COUNT(*) FROM demand_history;      -- 576 (4 regions × 6 SKUs × 24 months)
SELECT SUM(demand_share) FROM skus;       -- 1.00
```

Then, in Python or SQL, confirm the demand generator's output actually matches Lecture 1's stated region-level annual averages within a reasonable tolerance:

```sql
SELECT region_id, ROUND(AVG(monthly_total), 0) AS avg_monthly_units
FROM (
    SELECT region_id, month, SUM(units) AS monthly_total
    FROM demand_history
    GROUP BY region_id, month
) t
GROUP BY region_id
ORDER BY region_id;
```

## Expected outcome

| region_id | avg_monthly_units (expect within ±5%) |
|---|---:|
| MW | ~8,000 |
| NE | ~7,000 |
| SE | ~9,000 |
| WE | ~6,500 |

If any region is off by more than ~10%, check two things first: (1) did `SEASONAL_INDEX` actually get entered with all 12 months, and (2) did `SKU_SHARE` values sum to 1.00 (a typo here silently scales every region's total).

## Deliverable

A directory `exercise-01/` containing `schema.sql`, `generate_demand.py`, and `verification.md` with every check's actual output pasted in next to its expected value.

## Rules

- Do not alter any of Lecture 1's cost, capacity, or SKU-share numbers — the whole week's numeric consistency (including the quiz answer key) depends on this exact seed data.
- The demand generator must use `np.random.seed(40)` — this is what makes your numbers reproducible and comparable to everyone else taking this course.
