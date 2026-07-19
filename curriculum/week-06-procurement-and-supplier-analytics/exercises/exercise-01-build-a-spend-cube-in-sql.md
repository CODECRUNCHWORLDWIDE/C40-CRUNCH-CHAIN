# Exercise 1 — Build a Spend Cube in SQL

**Time:** ~1.5 hours. **Builds on:** Lecture 1 — Spend Analysis & the Spend Cube.

Use the `purchase_orders` table set up in this week's [README](../README.md). Write every query yourself before checking the expected output.

## Task 1 — Spend by month

Compute total spend per calendar month, ordered chronologically.

```sql
-- PostgreSQL
SELECT
    DATE_TRUNC('month', order_date)::date AS spend_month,
    ROUND(SUM(qty_ordered * unit_price), 2) AS monthly_spend
FROM purchase_orders
GROUP BY spend_month
ORDER BY spend_month;

-- SQLite
SELECT
    strftime('%Y-%m-01', order_date) AS spend_month,
    ROUND(SUM(qty_ordered * unit_price), 2) AS monthly_spend
FROM purchase_orders
GROUP BY spend_month
ORDER BY spend_month;
```

**Expected:** three rows (January, February, March 2026). January and February should each be well over $250,000; write down which month is highest before moving on — you'll need it for Task 4.

## Task 2 — Category × business unit cross-tab

Build a two-dimensional cube: total spend for every (category, business_unit) combination that actually occurs in the data.

```sql
SELECT
    category,
    business_unit,
    ROUND(SUM(qty_ordered * unit_price), 2) AS spend
FROM purchase_orders
GROUP BY category, business_unit
ORDER BY category, spend DESC;
```

**Expected:** 7 rows. `Fabric` and `CMT` should each show up under more than one business unit — check which.

## Task 3 — The full cube with `GROUPING SETS`

Task 2 gave you one slice. A real spend cube usually needs the category-only totals, the business-unit-only totals, *and* the grand total, all in one result set — that's what `GROUPING SETS` is for. Write a single query that returns, in one result set:

- spend by `category` alone
- spend by `business_unit` alone
- the grand total (one row, both dimensions `NULL`)

```sql
SELECT
    category,
    business_unit,
    ROUND(SUM(qty_ordered * unit_price), 2) AS spend
FROM purchase_orders
GROUP BY GROUPING SETS (
    (category),
    (business_unit),
    ()
)
ORDER BY category NULLS LAST, business_unit NULLS LAST;
```

> **SQLite note:** `GROUPING SETS` isn't supported. Get the same result with three separate `SELECT`s combined by `UNION ALL` (project a literal `NULL` for the dimension not being grouped in each branch), or do this step in pandas with `pd.concat` of three `groupby` calls.

**Expected:** 5 category rows + 3 business-unit rows + 1 grand-total row = 9 rows. The grand total should read **$843,610.00** — if it doesn't match your Task-1 monthly sum total, you have a bug.

## Task 4 — Pareto and tail spend

Reproduce Lecture 1 Section 4's cumulative-percentage-by-supplier query, then answer in a comment above your query: **how many suppliers make up the first 80% of cumulative spend?**

```sql
WITH ranked AS (
    SELECT supplier, SUM(qty_ordered * unit_price) AS supplier_spend
    FROM purchase_orders
    GROUP BY supplier
)
SELECT
    supplier,
    ROUND(supplier_spend, 2) AS supplier_spend,
    ROUND(100.0 * SUM(supplier_spend) OVER (ORDER BY supplier_spend DESC)
          / SUM(supplier_spend) OVER (), 1) AS cumulative_pct
FROM ranked
ORDER BY supplier_spend DESC;
```

**Expected:** matches Lecture 1's table exactly — **4 suppliers** cross the 80% line (cumulative reaches 81.5% at the 4th supplier).

## Task 5 — Isolate maverick buying by category

Confirm which category all the flagged maverick spend lives in, and what fraction of *that category's* spend (not total spend) it represents.

```sql
SELECT
    category,
    ROUND(SUM(CASE WHEN maverick_buy THEN qty_ordered * unit_price ELSE 0 END), 2) AS maverick_spend,
    ROUND(SUM(qty_ordered * unit_price), 2) AS category_spend,
    ROUND(100.0 * SUM(CASE WHEN maverick_buy THEN qty_ordered * unit_price ELSE 0 END)
          / SUM(qty_ordered * unit_price), 1) AS pct_maverick_of_category
FROM purchase_orders
GROUP BY category
HAVING SUM(CASE WHEN maverick_buy THEN qty_ordered * unit_price ELSE 0 END) > 0;
```

**Expected:** one row, `MRO / Indirect`. Notice the percentage here is much higher than the 0.47%-of-total-spend figure from Lecture 1 — **100% of this category's spend is maverick.** That reframing (0.5% of the company vs. 100% of the category) is exactly the kind of number a category manager for MRO/Indirect would lead with when asking for a consolidation budget.

## Watch for

- **Grouping at the wrong grain.** `AVG(unit_price)` across PO lines is *not* the same as a volume-weighted average price (`SUM(qty*price)/SUM(qty)`) — a supplier with one huge order and one tiny order will look different depending on which you use. Lecture 1 and this exercise always use the volume-weighted version for spend totals; be deliberate about which one a given question actually needs.
- **NULLs from `GROUPING SETS` are structural, not missing data.** In Task 3, a `NULL` in the `business_unit` column of a category-only row doesn't mean the data is missing — it means "this row is a category subtotal, business unit doesn't apply." `ORDER BY ... NULLS LAST` keeps those subtotal/grand-total rows from sorting to the top by accident.

When done, keep your queries — Exercise 2 reuses this same table with a different `GROUP BY` grain.
