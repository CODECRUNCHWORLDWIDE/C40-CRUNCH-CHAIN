# Week 10 — Sales & Operations Planning (S&OP)

> **Goal:** by Sunday you can run a full monthly S&OP cycle end to end — pull demand from two competing sources into one consensus number, check it against real capacity, pick a production strategy and cost it out, and reconcile the whole thing against a finance revenue target — producing one balanced, defensible plan instead of three departments arguing past each other with three different spreadsheets.

Welcome to **C40 · Crunch Chain**, Week 10. Weeks 3–4 forecast demand. Weeks 5–6 sized inventory and picked suppliers. Weeks 7–8 moved and stored product. Week 9 optimized flow through the network with LP/MIP. Every one of those was a single function doing its job well. **S&OP is the process that makes those functions agree with each other.** Demand planning, supply/operations, and finance each keep their own number for "how much are we going to sell/make/earn next month" — and left alone, those three numbers drift apart until a stockout, a write-off, or a missed earnings call forces a painful reconciliation. S&OP is the standing monthly meeting (and, increasingly, the always-on process called **Integrated Business Planning**, or IBP) that catches the drift every month instead of once a year.

We work against a **six-month planning horizon** (January–June 2025) across Crunch Gear's three product families this week: **Trail Footwear**, **Backpacks & Bags**, and **Apparel**. Four seed tables carry the whole story — a demand-review table with two competing forecasts that must be reconciled into one, a supply-review table with regular/overtime/subcontract capacity and cost, a small reference table of prices and inventory, and a finance table with the revenue target Finance built independently, before this month's numbers existed. That gap between "what Finance assumed" and "what Demand and Supply now know" is the whole subject of Lecture 3.

**Data rule for this course:** every plan, every balance table, every cost comparison this week is built in **SQL (PostgreSQL 16, SQLite fallback)** and/or **Python (pandas)** — never a spreadsheet. S&OP has historically lived in Excel at most companies, and it's exactly the wrong tool for it: a spreadsheet can't join four data sources cleanly, can't recompute a running inventory balance without fragile drag-fill formulas, and leaves no audit trail of what changed between cycles. A relational table plus a window function does both, correctly, every time. Spreadsheets are covered separately in [C41 Crunch Excel](../../../C41-CRUNCH-EXCEL/).

## Learning objectives

By the end of this week, you will be able to:

- **Explain** the monthly S&OP cycle — demand review, supply review, reconciliation, executive sign-off — and how it has evolved into **Integrated Business Planning (IBP)**, a continuous process instead of a once-a-month meeting.
- **Build** an aggregate supply-demand balance table from forecast, capacity, and inventory data in SQL, using a running-total window function to track ending inventory (or backorder) month over month.
- **Compare** chase, level, and mixed aggregate-planning strategies on total cost — hiring/layoff cost, overtime premium, and inventory holding cost — and explain why the "obviously right" strategy usually isn't.
- **Reconcile** the operational plan (units, capacity, inventory) with the financial plan (revenue, margin) onto one set of numbers that both operations and finance can sign off on.
- **Run** gap analysis on a constrained plan — identify exactly which months and families are short, quantify the shortfall, and recommend specific, costed actions to close it.

## Prerequisites

- Comfortable with `SELECT`, `GROUP BY`, joins, and **window functions** (`SUM() OVER (... ORDER BY ...)`) in SQL — Weeks 2–7 of this course, or [C33 Crunch SQL](../../../C33-CRUNCH-SQL/) Weeks 4–5.
- Comfortable with basic pandas (`read_sql`, `groupby`, a simple month-by-month loop) — the chase/level/mixed comparison in Lecture 2 is a running simulation, easiest to write as a small iterative script, the same way Week 7's routing heuristics were.
- Week 3–4 forecasting concepts (baseline vs. statistical forecast) and Week 5 safety-stock concepts (why you hold a buffer) are assumed but not required verbatim — this week reintroduces exactly what it needs from each.
- PostgreSQL 16+ **or** SQLite 3.35+, plus Python 3.10+ with `pandas`. See [`resources.md`](./resources.md) for install steps.

## Setup — seed the Week 10 S&OP tables

Everything this week runs against four small tables. Create all four once, before Lecture 1.

**PostgreSQL:**

```bash
createdb crunch_chain_wk10
psql crunch_chain_wk10
```

**SQLite:**

```bash
sqlite3 crunch_chain_wk10.db
```

### Table 1 — `demand_plan` (the demand review)

Two independently-built forecasts per product family per month — a **statistical forecast** (time-series baseline, the kind Week 3–4 produced) and a **sales input** (the field's read on the same month, informed by deals, trade shows, and gut feel) — plus the **consensus forecast**: the single number Demand Planning and Sales actually agreed to after debating the gap. Run this on either engine:

```sql
CREATE TABLE demand_plan (
    month                   DATE    NOT NULL,
    product_family          TEXT    NOT NULL,
    stat_forecast_units     INTEGER NOT NULL,   -- statistical baseline forecast
    sales_input_units       INTEGER NOT NULL,   -- sales/field forecast for the same month
    consensus_forecast_units INTEGER NOT NULL,  -- the ONE number the demand review agreed on
    PRIMARY KEY (month, product_family)
);

INSERT INTO demand_plan VALUES
('2025-01-01','Trail Footwear',8000,8200,8200),
('2025-02-01','Trail Footwear',8400,8600,8600),
('2025-03-01','Trail Footwear',10000,10400,10400),
('2025-04-01','Trail Footwear',12200,13400,12800),
('2025-05-01','Trail Footwear',13800,14200,14200),
('2025-06-01','Trail Footwear',12800,13000,13000),
('2025-01-01','Backpacks & Bags',5000,5100,5100),
('2025-02-01','Backpacks & Bags',5300,5400,5400),
('2025-03-01','Backpacks & Bags',6600,6800,6800),
('2025-04-01','Backpacks & Bags',8200,9600,8900),
('2025-05-01','Backpacks & Bags',10000,10200,10200),
('2025-06-01','Backpacks & Bags',9400,9600,9600),
('2025-01-01','Apparel',11000,11200,11200),
('2025-02-01','Apparel',10600,10800,10800),
('2025-03-01','Apparel',9400,9600,9600),
('2025-04-01','Apparel',7200,7400,7400),
('2025-05-01','Apparel',5900,6100,6100),
('2025-06-01','Apparel',5600,5800,5800);
```

Look closely at **Trail Footwear, April**: `stat_forecast_units = 12200`, `sales_input_units = 13400` — a 1,200-unit gap (Sales is banking on a new retail-partner launch the statistical model has no history for), and `consensus_forecast_units = 12800`, roughly the midpoint. That's not a rounding artifact — it's a **documented negotiation**, exactly the kind Lecture 1 walks through.

### Table 2 — `supply_plan` (the supply review)

Regular, overtime, and subcontract capacity available each month per family, and what each costs per unit:

```sql
CREATE TABLE supply_plan (
    month                     DATE    NOT NULL,
    product_family            TEXT    NOT NULL,
    regular_capacity_units    INTEGER NOT NULL,
    overtime_capacity_units   INTEGER NOT NULL,  -- MAX extra units available via overtime
    subcontract_capacity_units INTEGER NOT NULL, -- MAX extra units available via subcontractor
    unit_cost_regular         NUMERIC NOT NULL,
    unit_cost_overtime        NUMERIC NOT NULL,
    unit_cost_subcontract     NUMERIC NOT NULL,
    PRIMARY KEY (month, product_family)
);

INSERT INTO supply_plan VALUES
('2025-01-01','Trail Footwear',8500,850,0,22.00,31.00,0.00),
('2025-02-01','Trail Footwear',8500,850,0,22.00,31.00,0.00),
('2025-03-01','Trail Footwear',9500,950,500,22.00,31.00,34.00),
('2025-04-01','Trail Footwear',9500,950,800,22.00,31.00,34.00),
('2025-05-01','Trail Footwear',11000,1100,1200,22.50,31.50,35.00),
('2025-06-01','Trail Footwear',11000,1100,1000,22.50,31.50,35.00),
('2025-01-01','Backpacks & Bags',5500,550,0,16.00,22.40,0.00),
('2025-02-01','Backpacks & Bags',5500,550,0,16.00,22.40,0.00),
('2025-03-01','Backpacks & Bags',7000,700,300,16.50,23.10,26.00),
('2025-04-01','Backpacks & Bags',8500,850,600,16.50,23.10,26.00),
('2025-05-01','Backpacks & Bags',10500,1050,700,17.00,23.80,26.00),
('2025-06-01','Backpacks & Bags',10000,1000,500,17.00,23.80,26.00),
('2025-01-01','Apparel',11000,300,0,20.00,29.00,0.00),
('2025-02-01','Apparel',10600,300,0,20.00,29.00,0.00),
('2025-03-01','Apparel',9400,250,0,20.00,29.00,0.00),
('2025-04-01','Apparel',7200,200,0,20.00,29.00,0.00),
('2025-05-01','Apparel',5900,150,0,20.00,29.00,0.00),
('2025-06-01','Apparel',5600,150,0,20.00,29.00,0.00);
```

Trail Footwear and Backpacks & Bags share a cutting-and-sewing line with limited **subcontract** overflow capacity; Apparel runs on a separate, lower-volume line with no subcontract option at all (`subcontract_capacity_units = 0` every month) — that's a real constraint, not a data gap.

### Table 3 — `product_reference`

One row per family: price, standard cost, where inventory starts, and the safety-stock floor:

```sql
CREATE TABLE product_reference (
    product_family            TEXT PRIMARY KEY,
    unit_price                 NUMERIC NOT NULL,
    standard_unit_cost         NUMERIC NOT NULL,
    beginning_inventory_jan    INTEGER NOT NULL,  -- units on hand Jan 1, 2025
    safety_stock_target        INTEGER NOT NULL,  -- minimum ending inventory, every month
    holding_cost_per_unit_month NUMERIC NOT NULL,
    gross_margin_target_pct    NUMERIC NOT NULL
);

INSERT INTO product_reference VALUES
('Trail Footwear',145.00,62.00,1500,900,3.00,55.0),
('Backpacks & Bags',95.00,38.00,1100,700,2.25,58.0),
('Apparel',120.00,48.00,2600,1200,2.75,57.0);
```

Apparel starts the year sitting on **2,600 units** against a 1,200-unit target — 1,400 units of excess carried over from the holiday season. That's not a bug either; it's this week's inventory-drawdown story.

### Table 4 — `financial_targets` (the annual operating plan, by month)

The revenue number Finance built during last quarter's budget cycle — **before** this month's demand review happened:

```sql
CREATE TABLE financial_targets (
    month           DATE    NOT NULL,
    product_family  TEXT    NOT NULL,
    revenue_target  NUMERIC NOT NULL,
    PRIMARY KEY (month, product_family)
);

INSERT INTO financial_targets VALUES
('2025-01-01','Trail Footwear',1100000),('2025-02-01','Trail Footwear',1200000),
('2025-03-01','Trail Footwear',1400000),('2025-04-01','Trail Footwear',1700000),
('2025-05-01','Trail Footwear',1950000),('2025-06-01','Trail Footwear',1900000),
('2025-01-01','Backpacks & Bags',460000),('2025-02-01','Backpacks & Bags',500000),
('2025-03-01','Backpacks & Bags',620000),('2025-04-01','Backpacks & Bags',800000),
('2025-05-01','Backpacks & Bags',970000),('2025-06-01','Backpacks & Bags',910000),
('2025-01-01','Apparel',1300000),('2025-02-01','Apparel',1250000),
('2025-03-01','Apparel',1100000),('2025-04-01','Apparel',850000),
('2025-05-01','Apparel',700000),('2025-06-01','Apparel',680000);
```

Sanity checks — these should print `18`, `18`, `3`, `18`:

```sql
SELECT COUNT(*) FROM demand_plan;
SELECT COUNT(*) FROM supply_plan;
SELECT COUNT(*) FROM product_reference;
SELECT COUNT(*) FROM financial_targets;
```

## Weekly schedule

Adds up to roughly the course's **~28 hr/week full-time pace**.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|--------------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | The S&OP cycle; demand + supply review | 2h | 0h | 0h | 0.5h | 1h | 0h | 3.5h |
| Tuesday | Balance table in SQL (window functions) | 0h | 1.5h | 0h | 0.5h | 1h | 0h | 3h |
| Wednesday | Aggregate planning — chase/level/mixed | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Thursday | IBP — finance reconciliation; gap analysis | 2h | 1h | 1h | 0.5h | 1h | 1h | 6.5h |
| Friday | Constrained plan + scenario planning challenges | 0h | 0h | 2h | 0.5h | 1h | 1.5h | 5h |
| Saturday | Mini-project — one full S&OP cycle | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **4h** | **4h** | **3h** | **3.5h** | **5h** | **5h** | **27.5h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-the-sop-process.md](./lecture-notes/01-the-sop-process.md) | The monthly S&OP cycle — demand review, supply review, reconciliation, exec sign-off — and why one set of numbers matters | 2h |
| 2 | [lecture-notes/02-aggregate-planning-and-balancing.md](./lecture-notes/02-aggregate-planning-and-balancing.md) | Chase vs. level vs. mixed strategies, capacity/inventory buffers, balancing supply to demand at the aggregate level | 2h |
| 3 | [lecture-notes/03-integrated-business-planning.md](./lecture-notes/03-integrated-business-planning.md) | IBP — linking the operational plan to finance, scenario planning, gap-closing actions | 2h |
| 4 | [exercises/exercise-01-build-a-supply-demand-table.md](./exercises/exercise-01-build-a-supply-demand-table.md) | Build the running supply-demand balance table in SQL with a window function | 1.5h |
| 5 | [exercises/exercise-02-compare-aggregate-plans.md](./exercises/exercise-02-compare-aggregate-plans.md) | Cost out chase, level, and mixed strategies for Trail Footwear in Python | 1.5h |
| 6 | [exercises/exercise-03-gap-analysis-and-actions.md](./exercises/exercise-03-gap-analysis-and-actions.md) | Find every month/family that breaches safety stock and cost the fix | 1h |
| 7 | [challenges/challenge-01-constrained-sop-plan.md](./challenges/challenge-01-constrained-sop-plan.md) | Build a feasible, minimum-cost 6-month plan for Trail Footwear that never stocks out | 2h |
| 8 | [challenges/challenge-02-scenario-planning-model.md](./challenges/challenge-02-scenario-planning-model.md) | Build an upside/downside scenario model and recommend which one to plan to | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Run one full S&OP cycle across all three families and ship a signed-off plan | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice tying demand, supply, and finance together | 5h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official references, glossaries, tools to install | — |

## By the end of this week you can…

- Walk into a monthly S&OP meeting and explain, in order, what demand review, supply review, reconciliation, and exec sign-off each produce.
- Write a SQL query that turns four raw tables into a running, auditable supply-demand balance table — no manual drag-fill, no version-mismatched tabs.
- Look at a demand spike against a capacity ceiling and know, with numbers, whether chase, level, or mixed is the cheaper way to meet it.
- Take a $-denominated finance target and a units-denominated ops plan and reconcile them onto the same page, including saying exactly where they disagree and why.
- Turn "we're going to miss a month" into a specific, costed, three-option recommendation instead of a shrug.

## Up next

Week 11 — Risk, resilience, and the digital supply chain: stress-testing the plan you just balanced against a disruption, and automating the replan when it breaks.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
