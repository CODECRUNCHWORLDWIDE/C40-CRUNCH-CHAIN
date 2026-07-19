# Week 2 — Quiz

Fifteen questions. Lectures closed. Aim for 12/15 before starting Week 3. A mix of multiple-choice and short "what does this return" questions — the answer key at the bottom explains the *why*, not just the letter.

---

**Q1.** In this week's schema, why are `plants`, `dcs`, and `stores` modeled as one `sites` table with a `site_type` column, rather than three separate tables?

- A) Postgres doesn't support more than 8 tables in a schema.
- B) They share the same columns and differ only by role; a `CHECK` constraint on `site_type` still enforces which roles are valid, without duplicating the plant/DC/store structure three times.
- C) `site_type` is required by SQL syntax for every table.
- D) There's no real reason — it's just a style preference with no consequence.

---

**Q2.** The `lanes` table's `one_origin_only` `CHECK` constraint enforces that:

- A) Every lane must have both an `origin_supplier_id` and an `origin_site_id` filled in.
- B) Exactly one of `origin_supplier_id` or `origin_site_id` is non-`NULL` — never both, never neither.
- C) A lane's destination must be a supplier, not a site.
- D) Lane costs must always be positive.

---

**Q3.** Why does `inventory_transactions` store a ledger of signed events instead of one `on_hand_qty` column updated in place on every movement?

- A) A ledger is always faster to query than a single balance column.
- B) SQL cannot express `UPDATE` statements.
- C) A ledger preserves full history, so any past point-in-time balance can be computed by summing up to that date — a single updated column loses that history and can silently desync from reality.
- D) There's no difference; both approaches store identical information.

---

**Q4.** You write `SELECT o.order_id, ol.qty_ordered, sl.qty_shipped FROM order_lines ol JOIN orders o ON ... LEFT JOIN shipment_lines sl ON ...` for an order line that hasn't shipped yet. What does `sl.qty_shipped` return for that row?

- A) `0`
- B) An empty string
- C) `NULL`
- D) The query errors out

---

**Q5.** Given Q4's answer, why is `SUM(sl.qty_shipped)` dangerous to use directly in a further calculation without `COALESCE(sl.qty_shipped, 0)` first?

- A) `SUM()` treats `NULL` as `0` automatically, so there's no danger.
- B) `SUM()` silently skips `NULL` values, which is usually fine for the sum itself — but any calculation that assumes every row contributed a real number (like a row count check) can be misled.
- C) `SUM()` raises an error whenever it encounters a `NULL`.
- D) `NULL` values cause `SUM()` to return `NULL` for the entire query, with no warning.

---

**Q6.** An order has 2 order lines and shipped in 2 separate shipments. You write `SELECT * FROM orders o JOIN shipments sh ON sh.order_id = o.order_id JOIN order_lines ol ON ol.order_id = o.order_id WHERE o.order_id = <that order>`. How many rows does this return for that one order?

- A) 1
- B) 2
- C) 4
- D) 0, because the join is invalid

---

**Q7.** What's the correct fix for Q6's problem, if you need order-line-level detail alongside shipment-level detail?

- A) Add `DISTINCT` to the `SELECT` clause.
- B) Route the join through `shipment_lines`, which correctly pairs each specific order line with the specific shipment that carried it, instead of joining `shipments` and `order_lines` to `orders` independently.
- C) Remove the `order_lines` join entirely.
- D) Switch `JOIN` to `LEFT JOIN` everywhere.

---

**Q8.** What is the key difference between what `GROUP BY` does and what a window function (`... OVER (...)`) does to the rows passed into it?

- A) They're two names for the exact same feature.
- B) `GROUP BY` collapses multiple rows into one row per group; a window function computes a value per row while keeping every original row intact.
- C) Window functions can only be used with `COUNT()`.
- D) `GROUP BY` keeps every row; window functions collapse them.

---

**Q9.** In `SUM(qty_change) OVER (PARTITION BY sku_id, site_id ORDER BY txn_date)`, what does `PARTITION BY` do?

- A) Physically splits the table into separate files on disk.
- B) Resets the running calculation independently for each distinct `(sku_id, site_id)` combination, similar to what `GROUP BY` does for an aggregate — except the individual rows aren't collapsed.
- C) Deletes rows that don't match the partition.
- D) Sorts the entire result set globally, ignoring `sku_id` and `site_id`.

---

**Q10.** Why would an operations team promise a delivery time based on `PERCENTILE_CONT(0.9)` of historical transit days rather than `AVG()`?

- A) `PERCENTILE_CONT` is always a smaller number than `AVG()`.
- B) `AVG()` can be dragged around by a small number of outliers; a P90 tells you the value that 90% of shipments beat, which is a more honest number to promise against if you want to be right most of the time.
- C) `AVG()` cannot be computed on date differences.
- D) They're mathematically identical for any dataset.

---

**Q11.** `RANK()` and `DENSE_RANK()` differ in that:

- A) `RANK()` cannot handle ties at all; `DENSE_RANK()` can.
- B) `RANK()` skips the next rank number(s) after a tie; `DENSE_RANK()` assigns the next consecutive integer with no gap.
- C) They are two names for the same function.
- D) `DENSE_RANK()` only works on numeric columns.

---

**Q12.** `LAG(x) OVER (PARTITION BY customer ORDER BY order_date)` for a customer's very first order in the dataset returns:

- A) `0`
- B) The value from that customer's most recent order instead
- C) `NULL`, because there is no prior row in that customer's partition
- D) An error

---

**Q13.** In the SQL-to-pandas workflow this week, which statement is correct?

- A) pandas should hold the one true, lasting copy of any record that matters — SQL is just for initial loading.
- B) SQL is the system of record; pandas is a workbench for analysis that SQL genuinely can't do (modeling, charting), and anything worth keeping gets written back to SQL.
- C) It doesn't matter which tool holds the lasting data, as long as one of them does.
- D) pandas and SQL should each independently maintain their own separate copy of every table, kept manually in sync.

---

**Q14.** `to_sql("my_table", engine, if_exists="replace")`, run on a schedule (e.g., a weekly cron job), is risky because:

- A) It's always slower than `if_exists="append"`.
- B) It drops and recreates the table from the current DataFrame's shape every time it runs, silently destroying any history that isn't in that specific DataFrame — appropriate for a one-time setup, dangerous as a recurring habit.
- C) `"replace"` doesn't actually work in pandas; only `"append"` and `"fail"` are real options.
- D) It requires manually dropping the table first, so it can never run unattended.

---

**Q15.** Two independent calculations of the same KPI — one in SQL, one in pandas — disagree, even though both queries "look correct" at a glance. What's the single most likely root cause to check first, based on this week's lectures and challenges?

- A) A hardware failure in the database server.
- B) A join or merge fan-out multiplying rows somewhere in one of the two paths, inflating a `SUM()` on the "many" side of a one-to-many relationship.
- C) SQL and pandas use fundamentally incompatible arithmetic, so small disagreements are always expected and can be ignored.
- D) `pandas.read_sql` rounds all numbers to two decimal places automatically.

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **B** — one table with a type discriminator column avoids duplicating identical structure three times, while the `CHECK` constraint still enforces which roles are valid.
2. **B** — the XOR-style constraint: exactly one origin type, never both, never neither.
3. **C** — a ledger you sum on read preserves full history; an in-place balance column only ever tells you "now," and can silently drift from correct if any update is missed.
4. **C** — an unmatched `LEFT JOIN` row returns `NULL` for every column from the unmatched side, not `0` and not an error.
5. **B** — `SUM()` quietly ignores `NULL`s (which is often the desired behavior for the sum itself), but that quiet skipping is exactly what makes it easy to build a downstream calculation that wrongly assumes every row had a real value.
6. **C** — 4 rows: 2 order lines × 2 shipments, because both joins go independently through `order_id` with no bridge connecting the *specific* line to the *specific* shipment that carried it. This is the fan-out.
7. **B** — routing through `shipment_lines` (which stores the true `order_line_id` ↔ `shipment_id` pairing) eliminates the spurious combinations Q6's naive join creates.
8. **B** — this is the core distinction the whole lecture builds on: `GROUP BY` collapses; window functions preserve every row while still computing an aggregate-like value per row.
9. **B** — `PARTITION BY` is to window functions what `GROUP BY` is to aggregates: it resets the calculation per group, but the rows themselves survive in the output.
10. **B** — P90 tells you the value 90% of your historical shipments were faster than or equal to; `AVG()` can look deceptively good (or bad) if a small number of outliers skew it.
11. **B** — `RANK()` leaves a gap in the rank sequence after ties (1,1,3); `DENSE_RANK()` doesn't (1,1,2).
12. **C** — there's no row before the first one in that partition, so `LAG()` correctly returns `NULL` there.
13. **B** — SQL stays the system of record; pandas is a scratch workbench for what SQL genuinely can't do, and results worth keeping flow back into SQL.
14. **B** — `"replace"` recreates the table from scratch every call; on a recurring schedule this silently discards any data not present in that run's DataFrame, which is why it belongs in one-time setup code, not a loop.
15. **B** — the single most common cause of two independently "correct-looking" calculations disagreeing is a fan-out inflating a sum on one of the two paths, exactly as Lecture 2 Section 1, Lecture 3 Section 4, and Challenge 2 all demonstrated.

</details>

**Scoring:** 12+ → start Week 3. 9–11 → re-read the lecture sections behind your misses, especially Lecture 2 Section 1 and Lecture 3 Section 4 if you missed the fan-out questions. <9 → re-read all three lectures from the top; joins, `GROUP BY`, and window functions are the load-bearing skills for every remaining week of this course.
