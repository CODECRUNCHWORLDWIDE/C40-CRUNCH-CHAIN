# Exercise 3 — Rolling Inventory with Window Functions

**Goal:** Build both window-function patterns from Lecture 2 yourself — a running inventory balance over time, and lead-time percentiles/ranks across shipments — until `OVER (...)`, `PARTITION BY`, and `PERCENTILE_CONT` stop feeling like magic incantations and start feeling like the right, obvious tool for "what happened over time" and "what's typical vs. what's an outlier."

**Estimated time:** 1.5 hours.

## Setup

Confirm the inventory ledger is loaded:

```sql
SELECT COUNT(*) FROM inventory_transactions;  -- must print 16
```

Create `solutions.sql`, one `-- Task N` block per task.

## Part A — Running inventory balance

### Task 1 — One SKU, one site

Write the running-balance query for `sku_id = 1` (Summit Shell Jacket) at `site_id = 5` (Newark DC), same shape as Lecture 2 Section 3. *(Expected balances in order: 300, 200, 350, 270, 265.)*

### Task 2 — Every SKU/site combination, one query

Extend Task 1 with `PARTITION BY sku_id, site_id` so the query returns a running balance for **every** SKU/site pair in the ledger at once, not just one. *(Expected: 16 rows total, grouped into 4 independent running sequences — SKU 1 at site 5 ending at 265, SKU 1 at site 3 ending at 220, SKU 3 at site 6 ending at 170, SKU 3 at site 4 ending at 210.)*

### Task 3 — Lowest ending balance

From Task 2's result, write a query (you can wrap Task 2 in a CTE) that returns only the **final** running balance per SKU/site pair — the balance after that pair's last transaction — ordered ascending. *(Hint: window functions don't let you `WHERE` on the window result directly in the same query level; either wrap in a CTE/subquery and filter there, or use `DISTINCT ON` in Postgres. Expected lowest: SKU 1 at Reno DC, ending at 220.)*

### Task 4 — Flag a stockout risk

For each SKU/site pair, compute total units shipped out in March (`SUM(qty_change)` where `txn_type = 'shipment'`, as a positive number) and compare it to the final running balance from Task 3. Which SKU/site pair has the **smallest ratio** of ending balance to units shipped that month — i.e., holds the least cushion relative to how fast it's moving? State the pair and the ratio in a comment.

## Part B — Lead-time percentiles and ranks

### Task 5 — Median and P90 transit time

Using `shipments`, compute `PERCENTILE_CONT(0.5)` and `PERCENTILE_CONT(0.9)` of `delivery_date - ship_date` across all shipments, same as Lecture 2 Section 4. *(Expected: P50 = 2 days, P90 = 3 days.)*

### Task 6 — Rank every shipment by speed

For every shipment, show `shipment_id`, `carrier`, `transit_days`, its `RANK()` from slowest to fastest, and its `NTILE(4)` quartile. *(Expected: exactly one shipment — Prairie Supply Co.'s order 6, on Heartland Carriers — sits alone in quartile 4 with a 4-day transit, the single slowest shipment in the dataset.)*

### Task 7 — Percentile by carrier

Repeat Task 5, but `GROUP BY carrier` this time (join `shipments` to itself isn't needed — `PERCENTILE_CONT` works fine alongside a `GROUP BY`, same as any other aggregate). Which carrier has the worst (highest) P90 transit time?

## Expected results (spot checks)

- Task 1 → 300, 200, 350, 270, 265.
- Task 2 → 16 rows, 4 partitions.
- Task 3 → lowest ending balance is SKU 1 at Reno DC (220).
- Task 5 → P50 = 2, P90 = 3.
- Task 6 → Heartland Carriers' order-6 shipment is the sole quartile-4 (slowest) shipment.

## Done when…

- [ ] All 7 tasks run and match the expected values above.
- [ ] You can state, in one sentence each, the difference between what `GROUP BY` and `PARTITION BY` do to the rows in front of them.
- [ ] Task 4's written answer names a specific SKU/site pair and a ratio, not just "it depends."

## Stretch

- Rewrite Task 2 using `AVG(qty_change) OVER (PARTITION BY sku_id, site_id ORDER BY txn_date ROWS BETWEEN 1 PRECEDING AND CURRENT ROW)` — a 2-row moving average of transaction size instead of a running sum. What does a moving average of `qty_change` even mean here, given `qty_change` mixes positive receipts and negative shipments? Is this a useful metric for this ledger, or a case of using a window function just because you can? Defend your answer in 2–3 sentences.
- Using `LAG()`, compute the number of days between each SKU/site pair's consecutive transactions. Which pair goes the longest stretch without any inventory movement at all?

## Submission

Commit `solutions.sql` to your portfolio under `c40-week-02/exercise-03/`.
