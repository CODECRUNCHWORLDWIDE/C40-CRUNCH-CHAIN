# Exercise 2 — Join Orders to Shipments

**Goal:** Build a correct fulfillment view across `orders`, `order_lines`, `shipment_lines`, and `shipments` — and deliberately break it once, on purpose, so you can *see* a join fan-out with your own eyes before it ever costs you a wrong number in a real report.

**Estimated time:** 1.5 hours.

## Setup

Confirm Exercise 1's seed is loaded:

```sql
SELECT COUNT(*) FROM orders, order_lines, shipments, shipment_lines;  -- must print 12*18*14*18 = 54432
```

*(That number is deliberately silly — it's a cross join with no `WHERE`, included only so a wrong seed load shows up as an obviously wrong number instead of a plausible-looking one. If it doesn't print exactly 54,432, re-check Exercise 1 before continuing.)*

Create a file `solutions.sql`, one `-- Task N` block per task.

## Tasks

### Task 1 — The base join

Select `order_id`, `customer_name`, `sku_code`, `category`, and `qty_ordered` for every order line, joining `order_lines` → `orders` → `skus`. *(Expected: 18 rows.)*

### Task 2 — Add shipped quantity, correctly

Extend Task 1 with a `qty_shipped` column, `LEFT JOIN`ed from `shipment_lines` on `order_line_id`, wrapped in `COALESCE(..., 0)`. *(Expected: still 18 rows — this dataset ships every order line in exactly one shipment, so this join doesn't fan out. Verify: `SUM(qty_ordered) = 1520`, `SUM(qty_shipped) = 1495`.)*

### Task 3 — Add shipment detail, through the correct bridge

Extend Task 2 by joining `shipments` — but reach it **through** `shipment_lines.shipment_id`, not directly from `orders`. Add `ship_date`, `delivery_date`, and `carrier`. *(Expected: still 18 rows. If you get more than 18, you joined `shipments` on the wrong key — go back and check you're joining on `shipment_lines.shipment_id = shipments.shipment_id`, not on `order_id`.)*

### Task 4 — See the fan-out on purpose

Now write the **wrong** version, deliberately, so you can see the failure mode Lecture 2 Section 1 described in the abstract:

```sql
-- WRONG on purpose -- do not use this pattern for real work
SELECT o.order_id, ol.order_line_id, sh.shipment_id
FROM orders o
JOIN shipments sh   ON sh.order_id = o.order_id
JOIN order_lines ol ON ol.order_id = o.order_id
WHERE o.order_id IN (3, 10);
```

Run it and count the rows for order 3 and order 10 separately. *(Expected: 4 rows for order 3, 4 rows for order 10 — each has 2 order lines and 2 shipments, and this join pairs every line with every shipment, giving 2×2 instead of the correct 2.)*

In `solutions.sql`, under this task, write 2–3 sentences: which specific join condition is missing that would fix this (hint: it's the same bridge column Task 3 used), and what the correct row count for order 3 should be.

### Task 5 — Per-order fulfillment summary

Build a query that returns one row per order with: `order_id`, `customer_name`, `promised_date`, `latest_delivery_date` (the `MAX` of that order's shipment delivery dates), `total_qty_ordered`, `total_qty_shipped`, `delivered_on_time` (boolean: `latest_delivery_date <= promised_date`), and `delivered_in_full` (boolean: `total_qty_shipped >= total_qty_ordered`).

*(Expected: 12 rows. Orders 1, 4, 5, 7, 8, 11, 12 should show `TRUE` for both booleans. Orders 2, 6, 9 should show `delivered_on_time = FALSE` but `delivered_in_full = TRUE`. Orders 3 and 10 should show `FALSE` for both.)*

### Task 6 — Which carrier moved the most product?

Using the correct bridge from Task 3, group by `carrier` and sum `qty_shipped`. Order results descending. *(Expected top carrier: **Regional Freight Co.**, with **475** total units shipped — more than any other carrier, driven by handling both Southeast orders that needed a second, corrective shipment.)*

## Expected results (spot checks)

- Task 1 → 18 rows.
- Task 2 → `SUM(qty_ordered) = 1520`, `SUM(qty_shipped) = 1495`.
- Task 4 → 4 rows each for orders 3 and 10 (the deliberately wrong query).
- Task 5 → orders 1, 4, 5, 7, 8, 11, 12 are `TRUE`/`TRUE` (7 orders — this is this dataset's OTIF count from Lecture 2).
- Task 6 → Regional Freight Co., 475 units.

## Done when…

- [ ] Tasks 1–3 and 5–6 all run and match the expected row counts / values above.
- [ ] Task 4's deliberately-wrong query is in your file, alongside your written explanation of the fix.
- [ ] You can explain, out loud, without looking at your notes, why `LEFT JOIN ... COALESCE(x, 0)` is different from `JOIN ... COALESCE(x, 0)` for Task 2's purpose.

## Stretch

- Rewrite Task 5 using a `WITH` clause (CTE) that pre-aggregates `shipment_lines` by `order_line_id` **before** joining it to `order_lines` — this is the "aggregate before you fan out" fix Lecture 2 Section 1 named. Confirm it produces identical results to your Task 5 answer.
- Add a `days_late` column to Task 5's output: `GREATEST(latest_delivery_date - promised_date, 0)`. Which order is latest, and by how many days?

## Submission

Commit `solutions.sql` to your portfolio under `c40-week-02/exercise-02/`.
