# Week 2 — Homework

Five problems, ~5 hours total, spread across the week. All problems query the exact database you seeded in Exercise 1 — if a query returns something unexpected, suspect your seed data before you suspect the problem. Commit each.

---

## Problem 1 — Twenty warm-up questions (45 min)

Short answers, one or two sentences each, in `warmups.md`. These check that Lecture 1–3's vocabulary is solid before you lean on it for the rest of the course.

1. What makes a database a "system of record" that a spreadsheet is not — name two specific reasons.
2. Why does this week's schema use one `sites` table with a `site_type` column instead of three separate tables (`plants`, `dcs`, `stores`)?
3. Explain the `one_origin_only` `CHECK` constraint on `lanes` in your own words — what bad data does it prevent?
4. What is the "header/line" pattern, and name the two pairs of tables in this week's schema that use it.
5. Why does `inventory_transactions` store signed events instead of a single `on_hand_qty` column that gets updated in place?
6. What's the difference between an `INNER JOIN` and a `LEFT JOIN`, in terms of which rows survive?
7. Why does `LEFT JOIN`ing to an unmatched row produce `NULL`, not `0` — and why does that matter for `SUM()`?
8. Define "join fan-out" in one sentence.
9. Name one concrete symptom that tells you a query result has been affected by a fan-out.
10. What does `GROUP BY` do to the rows it operates on that a window function does not?
11. Write, from memory, the general shape of a window function call (the keyword that distinguishes it from a plain aggregate).
12. What does `PARTITION BY` do inside a window function's `OVER (...)` clause?
13. Why is a percentile (like `PERCENTILE_CONT(0.9)`) often more useful than `AVG()` for setting a delivery-time promise?
14. What's the difference between `RANK()` and `DENSE_RANK()` when there's a tie?
15. What does `LAG()` let you compute that a plain `GROUP BY` cannot?
16. In the SQL-to-pandas workflow, what should *always* remain the system of record?
17. Name one kind of task that genuinely belongs in pandas rather than SQL, and why.
18. What does `to_sql(..., if_exists="append")` do differently from `if_exists="replace"` — and why is `"replace"` risky to run on a schedule?
19. Why does this week's `kpi_snapshots` example table use a long/tidy shape (`metric`, `value`, `period_start`, `period_end`) instead of one column per metric?
20. In your own words, why does cross-checking a KPI in both SQL and pandas catch bugs that computing it once, in either tool alone, would not?

---

## Problem 2 — Business-question query set (90 min)

Six real questions a Crunch Gear ops manager would actually ask, against your Exercise 1 database. Put each query and its result under a `-- Q` comment in `business-questions.sql`.

1. **Category mix.** Total `qty_ordered`, grouped by SKU `category`. *(A claim to check, not trust: "accessory is the highest-volume category, ahead of outerwear." Run the query — is that claim actually true? State the real ranking of all four categories and their totals.)*
2. **Best customer by volume.** Which customer ordered the most total units across all their orders? *(Expected: Ridgeline Retail, 500 units — the highest of the four customers.)*
3. **Lane economics.** Average `cost_per_unit`, grouped by `mode` (`truck`, `rail`, `ocean`). Which mode is cheapest per unit on average, and does that surprise you given `ocean` lanes also carry the longest `standard_transit_days`?
4. **Supplier exposure.** For each supplier, how many lanes originate from them, and what's the combined `lead_time_days` if a company sourced from all of them sequentially (i.e., just sum it, not anything more sophisticated)? What does a purely additive lead time like this over- or understate about real sourcing risk?
5. **DC workload.** Count of orders and total `qty_ordered`, grouped by `source_dc_id` (join to `sites` for a readable name). Which DC handles the most order volume?
6. **The a-few-line trap.** Write a query that returns, for each order, its `order_id` and its **number of order lines** (`COUNT(order_line_id)`). Which orders have more than one line? Cross-check this list against which orders needed more than one shipment in Exercise 2 — are they the same orders, or different? What does that tell you about whether "multi-line" and "multi-shipment" are the same risk factor or two separate ones?

*(Problem 2, Q1 contains a deliberately planted wrong number in its own hint — the task is testing whether you run the query and trust the database over a plausible-sounding sentence. State explicitly, in a comment, which number in the hint was wrong and what the correct one is.)*

---

## Problem 3 — Window-function practice on a new mini-ledger (60 min)

A new SKU/site ledger, not from Exercise 1 — paste this into a scratch table (or a CTE with `VALUES`) called `practice_ledger`:

```sql
CREATE TABLE practice_ledger (
    txn_id INTEGER, txn_date DATE, txn_type TEXT, qty_change INTEGER
);
INSERT INTO practice_ledger VALUES
(1, '2026-04-01', 'receipt',    200),
(2, '2026-04-03', 'shipment',   -60),
(3, '2026-04-05', 'shipment',   -45),
(4, '2026-04-09', 'receipt',    120),
(5, '2026-04-12', 'shipment',   -90),
(6, '2026-04-15', 'adjustment', -8),
(7, '2026-04-20', 'receipt',     75),
(8, '2026-04-24', 'shipment',  -110);
```

In `window-practice.sql`:

1. Compute the running balance after every transaction. *(Expected final balance: 82.)*
2. Compute the **lowest point** the running balance ever reached, and the date it happened. *(This is the "closest to a stockout" moment — a number a plain `SUM()` over the whole table could never surface, since the whole-table sum only tells you the *ending* balance, 82, not that the balance dipped lower than that in between.)*
3. Using `LAG()`, compute the number of days between each transaction and the one before it. What's the longest gap between any two consecutive transactions?

---

## Problem 4 — Explain the concepts (30 min)

In `concepts-writeup.md`, prose only, no more than 400 words total:

1. A teammate says: "Window functions are just `GROUP BY` but fancier." Correct them — explain the actual difference in a way a non-technical manager could follow, using one example from this week's data.
2. Explain, to someone who's never used a database, why `REFERENCES` (a foreign key) is a stronger guarantee than "we all agreed to always fill this column in correctly."
3. This week's schema makes a deliberate simplification: each SKU is produced at exactly one plant (`skus.plant_id` is a single, required column). Name one real business reason a company might need to relax this — a SKU produced at *two* plants — and sketch (in words, no SQL required) how the schema would have to change to allow it.

---

## Problem 5 — Small SQL-to-pandas round trip (45 min)

Using your Exercise 1 database:

1. In Python, `read_sql` a query that returns, per SKU, total `qty_ordered` and total revenue potential (`qty_ordered * unit_price`, joined from `skus`).
2. In pandas, compute each SKU's share of total revenue potential (its revenue ÷ the sum of all SKUs' revenue), sorted descending.
3. Write this result back to Postgres as a new table `sku_revenue_mix` with columns `sku_code`, `revenue_potential`, `revenue_share_pct`, using `to_sql(..., if_exists="replace")` — and in a one-sentence comment, justify why `"replace"` is the *correct* choice here specifically (tie your answer back to Lecture 3 Section 5's warning about it).
4. `read_sql` your own new `sku_revenue_mix` table back out, and print it, to confirm the round trip worked.

**Deliver** `revenue_mix.py` with all four steps and their printed output (as comments or a companion `revenue_mix_output.txt`).

---

## Time budget

| Problem | Time |
|--------:|----:|
| 1 | 45 min |
| 2 | 90 min |
| 3 | 60 min |
| 4 | 30 min |
| 5 | 45 min |
| **Total** | **~4.5 h** |

After homework, take the [quiz](./quiz.md) and ship the [mini-project](./mini-project/README.md).
