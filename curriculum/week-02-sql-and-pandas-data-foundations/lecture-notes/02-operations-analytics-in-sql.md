# Operations Analytics in SQL

You have a schema and it's seeded (Exercise 1). This lecture is where it starts paying off: joining across the whole chain to answer real questions, `GROUP BY` to roll individual rows up into business numbers, and window functions to compute two things a plain aggregate can't — a running balance over time, and a percentile. Every query below runs against Exercise 1's exact data, so the numbers in this lecture are real, checkable answers, not made-up placeholders.

## 1. Joining across the chain

Start with the simplest useful question: **for every order line, show the customer, the SKU, how much was ordered, and how much actually shipped.**

```sql
SELECT
    o.order_id,
    o.customer_name,
    s.sku_code,
    ol.qty_ordered,
    COALESCE(sl.qty_shipped, 0) AS qty_shipped
FROM order_lines ol
JOIN orders o        ON ol.order_id = o.order_id
JOIN skus s           ON ol.sku_id = s.sku_id
LEFT JOIN shipment_lines sl ON sl.order_line_id = ol.order_line_id
ORDER BY o.order_id, s.sku_code;
```

Two choices here matter more than they look:

- **`LEFT JOIN` to `shipment_lines`, not `JOIN`.** An `INNER JOIN` would silently drop any order line that hasn't shipped yet — exactly the rows a real operations analyst most needs to see (what's still owed to the customer). `LEFT JOIN` keeps every `order_lines` row and fills in `NULL` where no shipment exists yet.
- **`COALESCE(sl.qty_shipped, 0)`.** Once you `LEFT JOIN`, an unshipped line's `qty_shipped` comes back `NULL` — and `NULL` is not the same as `0` in SQL. `SUM()` silently skips `NULL`s, so any aggregate built on this column without `COALESCE` would understate the shortfall. This is the single most common silent bug in operations SQL: forgetting that a `LEFT JOIN`'s unmatched side is `NULL`, not zero.

### The fan-out problem

Now walk the join one step further — bring in shipment details:

```sql
SELECT o.order_id, s.sku_code, ol.qty_ordered, sl.qty_shipped, sh.ship_date, sh.carrier
FROM order_lines ol
JOIN orders o             ON ol.order_id = o.order_id
JOIN skus s                ON ol.sku_id = s.sku_id
JOIN shipment_lines sl     ON sl.order_line_id = ol.order_line_id
JOIN shipments sh          ON sl.shipment_id = sh.shipment_id
WHERE o.order_id = 3;
```

This returns **two rows** for order 3, not one — because order 3 shipped in two separate shipments (Exercise 1's seed deliberately split it: SKU-3 shipped complete on March 8th, SKU-7 shipped short and late on March 10th). That's correct here, because you asked for shipment-level detail. But it's a trap the moment you forget it: if you then wrote `SELECT order_id, SUM(qty_ordered) FROM (that join) GROUP BY order_id`, you'd double-count `qty_ordered` for order 3, because the join duplicated its `order_lines` rows once per matching shipment. This is called a **fan-out** — a join that legitimately multiplies rows, and it silently wrecks any aggregate computed *after* the join instead of *before* it. The fix, always: aggregate the "one" side of a one-to-many join **before** joining it to the "many" side, or aggregate carefully with `COUNT(DISTINCT ...)` / a pre-aggregated subquery. Exercise 2 makes you build exactly this correctly.

## 2. `GROUP BY` aggregation: rolling rows into answers

**Question: what's total unit fill rate, and how does it break out by region?**

```sql
SELECT
    o.region,
    SUM(ol.qty_ordered)                                   AS units_ordered,
    SUM(COALESCE(sl.qty_shipped, 0))                      AS units_shipped,
    ROUND(100.0 * SUM(COALESCE(sl.qty_shipped, 0))
                / SUM(ol.qty_ordered), 1)                 AS unit_fill_rate_pct
FROM order_lines ol
JOIN orders o                ON ol.order_id = o.order_id
LEFT JOIN shipment_lines sl  ON sl.order_line_id = ol.order_line_id
GROUP BY o.region
ORDER BY unit_fill_rate_pct;
```

Run this against Exercise 1's data and the Southeast region comes back lowest — both of this dataset's short-shipments (order 3's beanies, order 10's fleece vests) happened on Southeast orders. Overall, across all 1,520 ordered units, 1,495 shipped: a **98.4% unit fill rate**. That single high number is hiding both defects, which is exactly why "what's our fill rate" is the wrong question to stop at, and "what's our fill rate *by region*" is the right follow-up — `GROUP BY` is what makes asking the follow-up free instead of another afternoon of work.

**Question: what's OTIF, computed at the order level, where an order must be on time and fully shipped across *every* line and *every* shipment it took?**

This is harder than it looks, because "on time" and "in full" for an order with multiple shipments need to look at the *worst* shipment and the *sum* across lines, not any single row:

```sql
WITH order_status AS (
    SELECT
        o.order_id,
        MAX(sh.delivery_date) <= o.promised_date AS delivered_on_time,
        MIN(ol.qty_ordered) IS NOT NULL           -- always true; keeps the CTE self-documenting
            AND NOT EXISTS (
                SELECT 1 FROM order_lines ol2
                LEFT JOIN (
                    SELECT order_line_id, SUM(qty_shipped) AS shipped_total
                    FROM shipment_lines GROUP BY order_line_id
                ) agg ON agg.order_line_id = ol2.order_line_id
                WHERE ol2.order_id = o.order_id
                  AND COALESCE(agg.shipped_total, 0) < ol2.qty_ordered
            ) AS delivered_in_full
    FROM orders o
    JOIN order_lines ol   ON ol.order_id = o.order_id
    JOIN shipments sh     ON sh.order_id = o.order_id
    GROUP BY o.order_id, o.promised_date
)
SELECT
    ROUND(100.0 * SUM(CASE WHEN delivered_on_time AND delivered_in_full THEN 1 ELSE 0 END)
                / COUNT(*), 1) AS otif_pct
FROM order_status;
```

Running this against Exercise 1's seed gives **OTIF = 58.3%** (7 of 12 orders). That's a real, sobering number for a 98.4%-fill-rate business — proof of Week 1 Lecture 2's point that a business can look healthy on one KPI and be failing on another that customers actually feel: a fleece vest that arrives complete but two days late still breaks OTIF, even though it never touches fill rate.

Don't try to memorize this query — the point isn't the syntax, it's the pattern: **a `WITH` clause (a "common table expression," or CTE) lets you name an intermediate result and build on it**, instead of nesting subqueries five levels deep until nobody can read the query, including you, next week. Every CTE and correlated subquery you're not sure about, break it into pieces: run the inner `SELECT` alone first, look at what it returns, *then* wrap it in the next layer. Challenge 1 has you build a cleaner, fuller version of this report yourself.

## 3. Window functions, part 1: running inventory balance

A `GROUP BY` collapses many rows into one row per group — you lose the individual rows. Sometimes you want the opposite: keep every row, but attach a *running total across the rows before it*. That's what a **window function** does.

**Question: what was the Summit Shell Jacket's on-hand balance at the Newark DC after every inventory event in March?**

```sql
SELECT
    txn_date,
    txn_type,
    qty_change,
    SUM(qty_change) OVER (
        ORDER BY txn_date, txn_id
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_balance
FROM inventory_transactions
WHERE sku_id = 1 AND site_id = 5   -- CG-JKT-001 at Newark DC
ORDER BY txn_date, txn_id;
```

Result, straight from Exercise 1's ledger:

| txn_date | txn_type | qty_change | running_balance |
|---|---|--:|--:|
| 2026-03-01 | receipt | +300 | 300 |
| 2026-03-07 | shipment | -100 | 200 |
| 2026-03-15 | receipt | +150 | 350 |
| 2026-03-25 | shipment | -80 | 270 |
| 2026-03-29 | adjustment | -5 | 265 |

The syntax to internalize: `SUM(qty_change) OVER (ORDER BY txn_date, txn_id ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)`. `OVER (...)` is what makes this a **window function** instead of a plain aggregate — it tells Postgres "compute this `SUM` per row, over a window of other rows relative to it," instead of collapsing everything into one output row the way `GROUP BY` would. `ORDER BY txn_date, txn_id` defines the row order the running total accumulates in (the `txn_id` tiebreak matters — two transactions on the same date need a deterministic order or your running balance could differ between runs). `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` is the "window frame" — every row from the very first one up through the current one — which is exactly what a running total means. In Postgres, `ORDER BY` inside `OVER (...)` actually defaults to this same frame, so `SUM(qty_change) OVER (ORDER BY txn_date, txn_id)` alone would give you the identical result — but writing the frame explicitly, at least while it's new, makes it obvious what's happening instead of relying on a default you'd have to look up.

Compare this to what a `GROUP BY` version of the same question would look like — it can't be done. `GROUP BY sku_id, site_id` collapses all five rows into one, giving you only the *final* balance (265), with no way to see what the balance was on March 15th. A window function is the only tool in this lecture that can answer "what was true at every point along the way," which is exactly why it exists as a separate feature from aggregation, not a variant of it.

**PARTITION BY — multiple running balances in one query.** Add `PARTITION BY sku_id, site_id` and you get a running balance *per SKU per site*, computed independently, in a single query instead of one query per site:

```sql
SELECT
    sku_id, site_id, txn_date, txn_type, qty_change,
    SUM(qty_change) OVER (
        PARTITION BY sku_id, site_id
        ORDER BY txn_date, txn_id
    ) AS running_balance
FROM inventory_transactions
ORDER BY sku_id, site_id, txn_date;
```

`PARTITION BY` to a window function is what `GROUP BY` is to an aggregate — it resets the calculation at each new group — except the individual rows survive instead of collapsing. This one query produces four independent running balances (SKU-1 at Newark, SKU-1 at Reno, SKU-3 at Atlanta, SKU-3 at Columbus) side by side. Exercise 3 has you run this and use it to spot which site/SKU combination is closest to running out.

## 4. Window functions, part 2: lead-time percentiles

**Question: what's the median (P50) and 90th-percentile (P90) actual transit time across all shipments this month?**

```sql
SELECT
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY delivery_date - ship_date) AS p50_transit_days,
    PERCENTILE_CONT(0.9) WITHIN GROUP (ORDER BY delivery_date - ship_date) AS p90_transit_days
FROM shipments;
```

Against Exercise 1's 14 shipments, this returns **P50 = 2 days, P90 = 3 days**. `PERCENTILE_CONT` is technically an "ordered-set aggregate," not a window function proper — it still collapses to one row like `GROUP BY` would — but it belongs in the same conversation because it's solving the same class of problem: a plain `AVG()` would tell you the *mean* transit time, which a single four-day outlier (this dataset's late Prairie Supply shipment) can drag around; a percentile tells you what a "typical" and a "bad-case" shipment actually look like, which is what you plan safety stock and customer promise dates against — you promise against the P90, not the average, because you want to be right 90% of the time, not merely right "on average."

**Why is this framed alongside window functions and not just a plain `GROUP BY`?** Because the *window* form of the same idea is genuinely useful too — ranking each shipment against the pack, without collapsing rows:

```sql
SELECT
    shipment_id,
    delivery_date - ship_date AS transit_days,
    NTILE(4) OVER (ORDER BY delivery_date - ship_date) AS transit_quartile,
    RANK() OVER (ORDER BY delivery_date - ship_date DESC) AS slowest_rank
FROM shipments
ORDER BY transit_days DESC;
```

`NTILE(4) OVER (ORDER BY ...)` buckets every shipment into four roughly equal quartiles by transit time — quartile 4 is your slowest 25%, the ones worth investigating first. `RANK() OVER (ORDER BY ... DESC)` gives every shipment an explicit "how slow is this one, ranked against the others" number, with ties sharing a rank (two shipments both taking 4 days would both rank #1, and the next-slowest would jump straight to rank #3 — `RANK()` skips the number a tie "used up"; `DENSE_RANK()` is the variant that doesn't skip, worth knowing exists for when you want consecutive integers instead). Both queries keep every row visible, unlike the single collapsed `PERCENTILE_CONT` row above — that's the window-function value proposition in one sentence: **you get the aggregate-level insight without losing the ability to point at the specific row that's the problem.**

## 5. `LAG()` — comparing a row to the one before it

One more window function worth knowing this week, because it answers a question none of the above can: **did this order's units-per-shipment go up or down compared to this customer's previous order?**

```sql
SELECT
    o.customer_name,
    o.order_id,
    o.order_date,
    SUM(ol.qty_ordered) AS units_this_order,
    SUM(ol.qty_ordered) - LAG(SUM(ol.qty_ordered)) OVER (
        PARTITION BY o.customer_name ORDER BY o.order_date
    ) AS change_vs_prior_order
FROM orders o
JOIN order_lines ol ON ol.order_id = o.order_id
GROUP BY o.customer_name, o.order_id, o.order_date
ORDER BY o.customer_name, o.order_date;
```

`LAG(x) OVER (PARTITION BY ... ORDER BY ...)` reaches back to the *previous row in that partition's order* and pulls its value into the current row — here, each customer's previous order's total units, so you can subtract and see the swing. The first order for each customer necessarily returns `NULL` (there's nothing before it) — another `NULL` you'd need to `COALESCE` before feeding this into further arithmetic, same lesson as Section 1. `LAG()` (and its mirror, `LEAD()`, which looks *forward* instead of back) is the tool for exactly this shape of question — anything that compares "this row" to "the row before/after it" within some grouping — and it comes up constantly once you're looking for demand swings, which is precisely where Week 3's forecasting work picks this back up.

## 6. Recap: which tool for which question

| Question shape | Tool |
|---|---|
| "Show me detail across multiple tables" | `JOIN` (mind `LEFT` vs. inner, and fan-out) |
| "Collapse many rows into one number per group" | `GROUP BY` + aggregate (`SUM`, `COUNT`, `AVG`) |
| "A running total/balance, row by row, over time" | Window function (`SUM() OVER (ORDER BY ...)`) |
| "Independent running totals per category" | Window function + `PARTITION BY` |
| "A typical value and a bad-case value, ignoring outliers" | `PERCENTILE_CONT(...) WITHIN GROUP (...)` |
| "Rank or bucket rows without collapsing them" | `RANK()`, `DENSE_RANK()`, `NTILE()` — all window functions |
| "Compare this row to the previous/next one" | `LAG()` / `LEAD()` — window functions |

Every one of these ran, unedited, against Exercise 1's seed data — copy them into your own `psql` or `sqlite3` session and get the same numbers. Exercise 2 has you build the fulfillment view from Section 1 fully yourself; Exercise 3 has you build both window-function patterns from Sections 3–4 end to end.

**Next:** [Lecture 3 — The SQL-to-pandas Workflow](./03-sql-to-pandas-workflow.md).
