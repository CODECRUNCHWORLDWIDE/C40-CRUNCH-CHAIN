# Mini-Project — Load April's Raw Files, Then Produce a KPI Snapshot Query Pack

> Two parts, one deliverable. Part A: load a fresh batch of raw operational data — the kind that shows up as CSV exports from a real order/shipment system — into the schema you already built in Exercise 1, extending it correctly instead of rebuilding it. Part B: produce a full KPI snapshot query pack, in **SQL and pandas, cross-checked against each other**, that answers every one of Week 1's five KPIs (OTIF, fill rate, perfect order, inventory turns, cash-to-cash) plus a regional breakdown — for real, from the database, the way you were only able to sketch by hand in Week 1.

**Estimated time:** 4 hours.

This is the week's capstone, and it's deliberately built to feel like a real first week on an operations analytics team: someone hands you a folder of CSVs from "the system" and a request for a KPI snapshot by Friday. Nobody hands you a clean schema and a single INSERT script in real life — you get raw files, and the schema you already built either accommodates them or it doesn't.

---

## Deliverable

A directory in your portfolio `c40-week-02/mini-project/` containing:

1. `load-april.sql` **and/or** `load_april.py` — however you chose to get Part A's raw files into Postgres (SQL `COPY`/`INSERT`, or pandas `to_sql`).
2. `kpi-snapshot.sql` — the SQL query pack answering all of Part B.
3. `kpi_reconcile.py` — the pandas script that independently recomputes at least three of Part B's numbers and confirms they match the SQL answers.
4. `report.md` — every number, stated in plain language, with interpretation — plus a short section explicitly connecting this week's real numbers back to Week 1's hand-computed ones (see "Tying it together" below).

---

## Part A — Load April's raw files (90 min)

Below are April's raw order, shipment, and finance-snapshot data — the same shape a real system would hand you as a CSV export. These are **new rows for your existing schema from Exercise 1**, not a new database — Crunch Gear's suppliers, sites, lanes, and SKUs haven't changed since March, only the orders have. Load these as new rows into your already-running `orders`, `order_lines`, `shipments`, and `shipment_lines` tables, continuing the ID sequences from where Exercise 1 left off (order IDs 13–18, order line IDs 19–25, shipment IDs 15–20, shipment line IDs 19–25).

**April orders (`orders`):**

```csv
order_id,customer_name,region,order_date,promised_date,source_dc_id
13,TrailStop Outfitters,Northeast,2026-04-02,2026-04-09,5
14,Ridgeline Retail,Southeast,2026-04-03,2026-04-10,6
15,Prairie Supply Co.,Midwest,2026-04-05,2026-04-12,4
16,Summit & Sea,West,2026-04-06,2026-04-13,3
17,TrailStop Outfitters,Northeast,2026-04-10,2026-04-17,5
18,Ridgeline Retail,Southeast,2026-04-12,2026-04-19,6
```

**April order lines (`order_lines`):**

```csv
order_line_id,order_id,sku_id,qty_ordered
19,13,2,55
20,14,7,180
21,15,8,120
22,16,1,95
23,17,5,70
24,18,3,85
25,18,6,55
```

**April shipments (`shipments`):**

```csv
shipment_id,order_id,lane_id,ship_date,delivery_date,carrier
15,13,11,2026-04-07,2026-04-09,SwiftHaul Logistics
16,14,12,2026-04-09,2026-04-11,Regional Freight Co.
17,15,10,2026-04-10,2026-04-12,Heartland Carriers
18,16,9,2026-04-11,2026-04-13,Pacific Crest Trucking
19,17,11,2026-04-15,2026-04-17,SwiftHaul Logistics
20,18,12,2026-04-17,2026-04-18,Regional Freight Co.
```

**April shipment lines (`shipment_lines`):**

```csv
shipment_line_id,shipment_id,order_line_id,qty_shipped,damaged_in_transit,invoice_accurate
19,15,19,55,FALSE,TRUE
20,16,20,160,FALSE,TRUE
21,17,21,120,FALSE,TRUE
22,18,22,95,FALSE,TRUE
23,19,23,70,FALSE,TRUE
24,20,24,85,FALSE,TRUE
25,20,25,55,FALSE,TRUE
```

**A finance snapshot for the trailing twelve months** — this is new data, not per-order — create a small `financials` table for it:

```sql
CREATE TABLE financials (
    metric  TEXT PRIMARY KEY,
    value   NUMERIC NOT NULL
);
```

```csv
metric,value
annual_cogs,3100000
avg_inventory_value,510000
annual_net_sales,4650000
avg_accounts_receivable,380000
avg_accounts_payable,295000
```

Load all of the above however you prefer — `psql`'s `\copy` from actual `.csv` files you save from the blocks above, plain `INSERT` statements, or `pandas.read_csv` + `to_sql`. **Note one line shipped short and one line shipped late** in this batch (look closely at order 14's numbers) — don't "fix" the data, that's real, and Part B needs it.

**Verify the load:**

```sql
SELECT COUNT(*) FROM orders;          -- must now print 18 (12 from March + 6 new)
SELECT COUNT(*) FROM order_lines;     -- must now print 25
SELECT COUNT(*) FROM shipments;       -- must now print 20
SELECT COUNT(*) FROM shipment_lines;  -- must now print 25
SELECT COUNT(*) FROM financials;      -- must print 5
```

---

## Part B — Produce the KPI snapshot query pack (2 hours)

Using **all 18 orders now in the database** (March + April together), compute, in SQL:

1. **OTIF%** — at the order level, same definition as Exercise 2 Task 5 and Challenge 1.
2. **Unit fill rate, order fill rate, and perfect order rate** — same strict, four-dimension perfect order definition as Challenge 1.
3. **OTIF% by region** — all four regions.
4. **Inventory turns and DIO** — from the `financials` table, same formulas as Week 1 Lecture 2.
5. **Cash-to-cash cycle time** — DIO + DSO − DPO, from `financials`.

Then, in pandas, **independently recompute #1 and #2** — pull the raw detail with `pd.read_sql` (not the already-aggregated SQL result), do the aggregation in pandas, and confirm your two numbers match your SQL answers exactly. If they don't match, you have a fan-out bug — go find it before moving on, the same way Challenge 2 made you practice.

## Interpret the results in `report.md`

For each of the five numbers/breakdowns above: state the result and 1–2 sentences of interpretation — what it tells a manager, whether it looks healthy, and (for #3) which region needs the closest look and why.

---

## Tying it together

Close `report.md` with a short paragraph (100–150 words): compare this week's **real, database-computed** OTIF number to Week 1's **hand-computed** OTIF exercise numbers. They're not the same dataset, so the point isn't that the numbers should match — the point is the *process*. Name one concrete way that computing OTIF from a real relational schema (this week) is more trustworthy than computing it from a single flat table by hand (Week 1) — and one concrete risk (a fan-out, a missed `COALESCE`, a wrong join key) that the hand-computed version simply couldn't have had, because it never involved a join at all.

---

## Rules

- Part A must extend the existing schema, not create a second, parallel one — reuse `orders`/`order_lines`/`shipments`/`shipment_lines` from Exercise 1.
- Part B must be computed in **both** SQL and pandas for at least OTIF and fill rate, with both shown and reconciled — a single-tool answer is an incomplete submission this week, on purpose, since Lecture 3 and Challenge 2 both built toward this habit.
- No spreadsheet formulas anywhere in this project, per the course's data rule.
- Every number in `report.md` must be traceable to a query/script in your submission.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Part A — correct load | 20% | All five row-count checks pass; April's short-ship and late-delivery are preserved, not corrected |
| Part B — SQL correctness | 30% | All five numbers correct, verified against the full 18-order dataset |
| Part B — SQL/pandas reconciliation | 20% | OTIF and fill rate independently confirmed in pandas, matching SQL exactly |
| Interpretation | 15% | `report.md` explains what each number means, not just what it is |
| Tying it together | 10% | A specific, technical comparison drawn between this week's process and Week 1's, not a vague reflection |
| Tooling discipline | 5% | SQL + Python only, work shown, no spreadsheet formulas |

---

## Why this matters

This is the exact shape of a recurring Friday deliverable on a real operations analytics team: new raw data lands, you load it into a schema that was built to receive it, and you produce a trustworthy KPI snapshot — cross-checked, not just computed once and hoped correct. Keep `kpi-snapshot.sql` and `kpi_reconcile.py` — Week 3 pulls this same order history straight out of your database to build its first demand forecast, and having working, correct queries already in hand means you start that week from data, not from scratch.

When done: push, then take the [quiz](../quiz.md) and start [Week 3 — Demand forecasting fundamentals](../../week-03-demand-forecasting-fundamentals/).
