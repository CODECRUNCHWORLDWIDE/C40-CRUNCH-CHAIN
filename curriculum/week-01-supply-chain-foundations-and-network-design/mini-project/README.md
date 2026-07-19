# Mini-Project — Map a Real Supply Chain, Then Compute Its KPIs

> Two parts, one deliverable. Part A: pick a real supply chain and map it end to end, the way Exercise 1 had you practice. Part B: compute and interpret this week's five core KPIs from a supplied dataset — using **SQL and/or Python (pandas)**, not a spreadsheet, because that's how you'll work with operations data for the rest of this course.

**Estimated time:** 3 hours, best done Saturday after the exercises and challenges.

This is the week's capstone: prove you can do both halves of the job. Anyone can memorize "OTIF = on-time and in-full" — the mini-project checks whether you can (1) look at a real, messy business and correctly identify its nodes/flows/echelons, and (2) turn a raw table of order data into the exact numbers a manager would ask for, using real tooling instead of a formula typed once by hand.

---

## Deliverable

A directory in your portfolio `c40-week-01/mini-project/` containing:

1. `network-map.md` — Part A: your real-chain map (nodes, echelons, flows, decoupling point).
2. `kpi-analysis.sql` **or** `kpi_analysis.py` — Part B: the queries/code that compute all five KPIs from the seed data below. (Do both if you want the practice — pick at least one.)
3. `report.md` — Part B's results stated in plain language, with interpretation, plus one paragraph connecting Part A and Part B (see "Tying it together" below).

---

## Part A — Map a real supply chain (45–60 min)

Pick a real company or product **you have not already used** for Exercise 1 this week — go bigger and more ambitious this time; a company with public information about its operations (e.g., a well-known apparel, electronics, grocery, or furniture brand) works well because you can ground some of your assumptions in real, findable facts.

In `network-map.md`, using Lecture 1's vocabulary:

1. List its nodes end to end (raw material → customer), minimum 6, same as Exercise 1 but for a new subject.
2. Draw the echelon diagram with labeled lanes and at least one transportation mode per major leg.
3. Describe all three flows (product, information, cash) — this time, for at least one flow, cite or reference something you found (a news article about the company's supplier network, a factory location, a stated delivery promise) rather than inventing it from scratch.
4. Identify the decoupling point and justify it.
5. Name one place in this chain where you'd expect the bullwhip effect to show up, and why.

---

## Part B — Compute the KPIs from a supplied dataset (90–120 min)

Below is a self-contained SQL seed for a **larger, messier** quarter of Crunch Gear order data than anything you've seen this week — 20 orders across 4 regions, with damage and invoicing flags included so you can compute a *real* perfect order rate (not the simplified version from Exercise 2), plus a finance snapshot for inventory turns and cash-to-cash.

### Set up the seed data

**PostgreSQL:**

```bash
createdb crunch_chain
psql crunch_chain
```

**SQLite:**

```bash
sqlite3 crunch_chain.db
```

Then paste this into the shell (works unchanged on both engines):

```sql
CREATE TABLE order_lines (
    order_id            INTEGER PRIMARY KEY,
    customer            TEXT    NOT NULL,
    region              TEXT    NOT NULL,
    promised_date        DATE    NOT NULL,
    ship_date            DATE    NOT NULL,
    qty_ordered          INTEGER NOT NULL,
    qty_shipped          INTEGER NOT NULL,
    unit_price           NUMERIC NOT NULL,
    damaged_in_transit   BOOLEAN NOT NULL,
    invoice_accurate     BOOLEAN NOT NULL
);

INSERT INTO order_lines VALUES
(1, 'TrailStop Outfitters',  'Northeast', '2026-01-05','2026-01-05',150,150,42.00,FALSE,TRUE),
(2, 'TrailStop Outfitters',  'Northeast', '2026-01-07','2026-01-09',100,100,42.00,FALSE,TRUE),
(3, 'Ridgeline Retail',      'Southeast', '2026-01-08','2026-01-08',200,180,38.50,FALSE,TRUE),
(4, 'Ridgeline Retail',      'Southeast', '2026-01-10','2026-01-10', 90, 90,38.50,FALSE,FALSE),
(5, 'Prairie Supply Co.',    'Midwest',   '2026-01-12','2026-01-12',120,120,45.00,TRUE, TRUE),
(6, 'Prairie Supply Co.',    'Midwest',   '2026-01-14','2026-01-16', 80, 70,45.00,FALSE,TRUE),
(7, 'Summit & Sea',          'West',      '2026-01-15','2026-01-15',160,160,40.00,FALSE,TRUE),
(8, 'Summit & Sea',          'West',      '2026-01-18','2026-01-18',140,140,40.00,FALSE,TRUE),
(9, 'TrailStop Outfitters',  'Northeast', '2026-01-20','2026-01-20',110,100,42.00,FALSE,TRUE),
(10,'Ridgeline Retail',      'Southeast', '2026-01-22','2026-01-25',130,130,38.50,FALSE,TRUE),
(11,'Prairie Supply Co.',    'Midwest',   '2026-02-02','2026-02-02', 95, 95,45.00,FALSE,TRUE),
(12,'Summit & Sea',          'West',      '2026-02-04','2026-02-04',175,175,40.00,TRUE, TRUE),
(13,'TrailStop Outfitters',  'Northeast', '2026-02-06','2026-02-06',140,140,42.00,FALSE,TRUE),
(14,'Ridgeline Retail',      'Southeast', '2026-02-09','2026-02-09',100, 90,38.50,FALSE,TRUE),
(15,'Prairie Supply Co.',    'Midwest',   '2026-02-11','2026-02-14', 60, 60,45.00,FALSE,TRUE),
(16,'Summit & Sea',          'West',      '2026-02-13','2026-02-13',150,150,40.00,FALSE,TRUE),
(17,'TrailStop Outfitters',  'Northeast', '2026-02-16','2026-02-16',120,120,42.00,FALSE,FALSE),
(18,'Ridgeline Retail',      'Southeast', '2026-02-18','2026-02-18',200,200,38.50,FALSE,TRUE),
(19,'Prairie Supply Co.',    'Midwest',   '2026-02-20','2026-02-23', 85, 80,45.00,FALSE,TRUE),
(20,'Summit & Sea',          'West',      '2026-02-23','2026-02-23',130,130,40.00,FALSE,TRUE);

CREATE TABLE quarter_financials (
    metric  TEXT    PRIMARY KEY,
    value   NUMERIC NOT NULL
);

INSERT INTO quarter_financials VALUES
('annual_cogs',        2800000),
('avg_inventory_value',  460000),
('annual_net_sales',   4200000),
('avg_accounts_receivable', 345000),
('avg_accounts_payable',    260000);
```

Sanity check — this should print `20`:

```sql
SELECT COUNT(*) FROM order_lines;
```

**Note the added columns vs. this week's other tables:** `damaged_in_transit` and `invoice_accurate` let you compute a **real** perfect order rate (all four dimensions), not the simplified on-time+in-full version from Exercise 2.

### Compute all five KPIs

Using SQL (query the tables directly) and/or Python/pandas (load the tables with `pandas.read_sql` or export to CSV first), compute:

1. **OTIF%** — on-time and in-full, across all 20 orders.
2. **Fill rate** — unit fill rate, order fill rate, and line fill rate (they'll match here since each order is one line — say so).
3. **Perfect order rate** — on-time, in-full, damage-free, **and** invoice-accurate, all four required. This is the real version Exercise 2 only approximated.
4. **Inventory turns and DIO** — from `quarter_financials`, annualized.
5. **Cash-to-cash cycle time** — DIO + DSO − DPO, from `quarter_financials`.

Also compute, as a bonus cut that a real manager would ask for next:

6. **OTIF by region** — which of the 4 regions (Northeast, Southeast, Midwest, West) has the worst on-time-in-full performance? A `GROUP BY region` (SQL) or `.groupby('region')` (pandas) answers this in one line — this is exactly the kind of question a spreadsheet makes tedious and SQL/pandas make trivial, which is *why* this course never uses spreadsheets as a data store.

A minimal SQL starting point for #1 (you'll write the rest yourself, and #3/#6 need more than this shape):

```sql
SELECT
    ROUND(100.0 * SUM(CASE WHEN ship_date <= promised_date
                             AND qty_shipped >= qty_ordered
                        THEN 1 ELSE 0 END) / COUNT(*), 1) AS otif_pct
FROM order_lines;
```

### Interpret the results in `report.md`

For each of the six numbers above: state the result, and 1–2 sentences of plain-language interpretation (what it tells a manager, and whether it looks healthy). For #3 (perfect order), explicitly compare it to #1 (OTIF) and explain *why* they differ, using Lecture 2 §3's compounding logic.

---

## Tying it together

Close `report.md` with a short paragraph (100–150 words) connecting Part A and Part B: if the *real* company you mapped in Part A had order data that looked like Part B's — some late shipments, some short, a couple of damage/invoice issues — which node in your Part A map would you suspect is the most likely root cause of each problem type (late → probably which node? short → probably which node? damage → probably which lane?), and why?

---

## Rules

- Part B must be done in **SQL and/or Python** — no spreadsheet formulas, per this course's data rule (state which tool(s) you used in `report.md`).
- Show your query/code, not just the output — `kpi-analysis.sql` or `kpi_analysis.py` is a required deliverable, not optional.
- Every number in `report.md` must be traceable to a query/code cell in your SQL/Python file.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Part A — network map | 25% | 6+ correctly ordered nodes, clean diagram, all 3 flows addressed distinctly, justified decoupling point |
| Part B — correctness | 35% | All 6 numbers correct, computed in SQL/Python (verified against the dataset) |
| Part B — NULL/edge-case handling | 10% | Perfect order correctly requires all 4 dimensions; fill-rate variants correctly distinguished |
| Interpretation | 15% | `report.md` explains what each number means, not just what it is |
| Tying it together | 10% | A genuine, specific link drawn between Part A's map and Part B's failure patterns |
| Tooling discipline | 5% | SQL/Python only, work shown, no spreadsheet formulas |

---

## Why this matters

This is the shape of a real first assignment on an operations analytics team: someone hands you a network to understand and a table of order data to make sense of, and you're expected to produce both a clear map and clean, correct numbers — using the same tools (SQL, Python) you'll use every week for the rest of this course. Keep `kpi-analysis.sql`/`kpi_analysis.py`; Week 2 formalizes exactly this pattern — modeling and loading a full multi-echelon network into Postgres — and you'll be glad you already have working queries to build on.

When done: push, then take the [quiz](../quiz.md) and start [Week 2 — SQL & pandas data foundations for operations](../../week-02-sql-and-pandas-data-foundations/).
