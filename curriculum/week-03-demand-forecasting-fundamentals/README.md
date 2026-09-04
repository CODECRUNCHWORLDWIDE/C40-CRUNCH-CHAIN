# Week 3 — Demand Forecasting Fundamentals

> **Goal:** by Sunday you can pull two years of weekly demand for a SKU straight out of SQL, decompose it into trend/seasonality/noise, build three baseline forecasts and two smoothing forecasts, and say — with a number, not a feeling — which one actually deserves to go into production. You will never again ship a forecasting model without first beating a naive one.

Welcome to **C40 · Crunch Chain**, Week 3. Weeks 1–2 gave you the map (network structure, KPIs) and the fuel tank (SQL + pandas to hold real operations data). This week you start driving: **forecasting** is the input every other function in this course depends on. Inventory policy (Week 5) needs a forecast to size safety stock against. Procurement (Week 6) needs a forecast to time purchase orders. S&OP (Week 10) is *literally* the meeting where a forecast gets reconciled against supply and finance. Get this week wrong and every later number inherits the error.

We work against one running dataset all week: two years (104 weeks) of real weekly sell-through for six **Crunch Gear** SKUs — jackets, an accessory, a bag, a fleece, and a sandal — chosen because they disagree with each other on purpose. One peaks in winter, one peaks in summer, one has almost no seasonality at all, one is growing, one is aging out, and one gets hit with a promotional spike twice a year. Decompose all six and you've seen the shapes that show up in nearly every real retail and CPG demand series.

**Data rule for this course:** every table below lives in **SQL (PostgreSQL 16, SQLite fallback)** and gets pulled into **Python (pandas)** for the actual modeling and scoring. We never park a demand history in a spreadsheet — a spreadsheet can't hold a `GROUP BY`, can't be queried by five teammates at once, and doesn't scale from 6 SKUs this week to 50 in the mini-project to your employer's real catalog on the job. Whenever this course would traditionally reach for Excel, it reaches for `psycopg`/`sqlite3` + pandas instead, and says why in the lecture.

## Learning objectives

By the end of this week, you will be able to:

- **Decompose** a demand series into level, trend, seasonality, and noise — and know the difference between additive and multiplicative decomposition, and when each applies.
- **Build baseline forecasts** — naive, seasonal naive, and moving average — reading demand straight out of SQL into pandas, with no library beyond pandas doing the arithmetic.
- **Apply** single exponential smoothing (level only) and double exponential smoothing / Holt's method (level + trend), and explain what the smoothing parameters (α, β) actually control.
- **Measure forecast error honestly** with MAE, MAPE, RMSE, and bias — know what each metric is sensitive to, when MAPE lies to you, and what a positive vs. negative bias means operationally.
- **Always compare against a naive benchmark** — treat "beats naive" as the minimum bar for shipping any forecasting method, this week and every week after.
- **Explain, with a specific counter-example from this week's own data, why a simple well-tuned baseline can beat a fancier smoothing method** — and therefore why you score before you commit to a method.

## Standards this week meets

| Bar | What this week is measured against |
| --- | --- |
| University | `SCM 3301` — forecast demand with time-series methods, and measure forecast error honestly against a naive benchmark. |
| Industry | Ship a demand forecast for a live catalogue and defend its accuracy, in writing, to a planning manager who has never heard of MAPE. |
| Beyond the bar | Fifty SKUs rather than one tidy series, every method scored per SKU against the naive baseline, and a written finding when the simple method wins — `mini-project/README.md` |

## Prerequisites

- Comfortable with `SELECT`, `WHERE`, `ORDER BY`, `GROUP BY`, and basic aggregates in SQL (Week's 2 material, or [C33 Crunch SQL](../../../C33-CRUNCH-SQL/) Weeks 1–3).
- Comfortable reading a CSV/SQL result into a pandas `DataFrame` and doing basic column math (`df['x'] - df['y']`), sorting, and `groupby`.
- Python 3.10+ with `pandas` installed (`pip install pandas`). No forecasting library is required this week — you build every method by hand so you understand what's inside it before Week 4 lets a library do it for you.
- PostgreSQL 16+ **or** SQLite 3.35+, per the course setup. See [`resources.md`](./resources.md).

## Set up the seed data (do this first)

Everything this week runs against one table, `demand_history` — weekly unit sales for six SKUs, two full years, real enough to have real seasonality and real noise.

**PostgreSQL:**

```bash
createdb crunchchain
psql crunchchain
```

**SQLite:**

```bash
sqlite3 crunchchain.db
```

Create the schema (identical on both engines):

```sql
CREATE TABLE sku_dim (
    sku_id      TEXT PRIMARY KEY,
    sku_name    TEXT NOT NULL,
    category    TEXT NOT NULL
);

CREATE TABLE demand_history (
    sku_id      TEXT    NOT NULL REFERENCES sku_dim(sku_id),
    week_start  DATE    NOT NULL,      -- every Monday, ISO weeks
    units_sold  INTEGER NOT NULL,
    PRIMARY KEY (sku_id, week_start)
);

INSERT INTO sku_dim (sku_id, sku_name, category) VALUES
('JCK-ALP-001', 'Alpine Shell Jacket', 'Jackets'),
('JCK-STM-002', 'Summit Down Jacket',  'Jackets'),
('ACC-BEA-010', 'Merino Beanie',       'Accessories'),
('BAG-DAY-020', 'Daypack 22L',         'Bags'),
('FLC-ZIP-030', 'Fleece Half-Zip',     'Fleece'),
('SAN-TRL-040', 'Trail Sandal',        'Footwear');
```

Then load 624 weeks of demand (six SKUs × 104 weeks, 2023-01-02 through 2024-12-23):

```sql
-- JCK-ALP-001 (Alpine Shell Jacket) — 104 weeks
INSERT INTO demand_history (sku_id, week_start, units_sold) VALUES
('JCK-ALP-001','2023-01-02',164), ('JCK-ALP-001','2023-01-09',184), ('JCK-ALP-001','2023-01-16',186), ('JCK-ALP-001','2023-01-23',180),
('JCK-ALP-001','2023-01-30',188), ('JCK-ALP-001','2023-02-06',171), ('JCK-ALP-001','2023-02-13',175), ('JCK-ALP-001','2023-02-20',171),
('JCK-ALP-001','2023-02-27',167), ('JCK-ALP-001','2023-03-06',150), ('JCK-ALP-001','2023-03-13',166), ('JCK-ALP-001','2023-03-20',146),
('JCK-ALP-001','2023-03-27',144), ('JCK-ALP-001','2023-04-03',138), ('JCK-ALP-001','2023-04-10',116), ('JCK-ALP-001','2023-04-17',109),
('JCK-ALP-001','2023-04-24',113), ('JCK-ALP-001','2023-05-01',100), ('JCK-ALP-001','2023-05-08',92), ('JCK-ALP-001','2023-05-15',86),
('JCK-ALP-001','2023-05-22',88), ('JCK-ALP-001','2023-05-29',77), ('JCK-ALP-001','2023-06-05',59), ('JCK-ALP-001','2023-06-12',69),
('JCK-ALP-001','2023-06-19',70), ('JCK-ALP-001','2023-06-26',63), ('JCK-ALP-001','2023-07-03',75), ('JCK-ALP-001','2023-07-10',53),
('JCK-ALP-001','2023-07-17',59), ('JCK-ALP-001','2023-07-24',78), ('JCK-ALP-001','2023-07-31',69), ('JCK-ALP-001','2023-08-07',76),
('JCK-ALP-001','2023-08-14',79), ('JCK-ALP-001','2023-08-21',86), ('JCK-ALP-001','2023-08-28',89), ('JCK-ALP-001','2023-09-04',83),
('JCK-ALP-001','2023-09-11',114), ('JCK-ALP-001','2023-09-18',103), ('JCK-ALP-001','2023-09-25',113), ('JCK-ALP-001','2023-10-02',127),
('JCK-ALP-001','2023-10-09',136), ('JCK-ALP-001','2023-10-16',138), ('JCK-ALP-001','2023-10-23',132), ('JCK-ALP-001','2023-10-30',141),
('JCK-ALP-001','2023-11-06',164), ('JCK-ALP-001','2023-11-13',167), ('JCK-ALP-001','2023-11-20',178), ('JCK-ALP-001','2023-11-27',182),
('JCK-ALP-001','2023-12-04',173), ('JCK-ALP-001','2023-12-11',181), ('JCK-ALP-001','2023-12-18',203), ('JCK-ALP-001','2023-12-25',183),
('JCK-ALP-001','2024-01-01',178), ('JCK-ALP-001','2024-01-08',204), ('JCK-ALP-001','2024-01-15',186), ('JCK-ALP-001','2024-01-22',196),
('JCK-ALP-001','2024-01-29',172), ('JCK-ALP-001','2024-02-05',175), ('JCK-ALP-001','2024-02-12',180), ('JCK-ALP-001','2024-02-19',175),
('JCK-ALP-001','2024-02-26',173), ('JCK-ALP-001','2024-03-04',170), ('JCK-ALP-001','2024-03-11',161), ('JCK-ALP-001','2024-03-18',138),
('JCK-ALP-001','2024-03-25',154), ('JCK-ALP-001','2024-04-01',145), ('JCK-ALP-001','2024-04-08',130), ('JCK-ALP-001','2024-04-15',124),
('JCK-ALP-001','2024-04-22',98), ('JCK-ALP-001','2024-04-29',111), ('JCK-ALP-001','2024-05-06',110), ('JCK-ALP-001','2024-05-13',92),
('JCK-ALP-001','2024-05-20',93), ('JCK-ALP-001','2024-05-27',81), ('JCK-ALP-001','2024-06-03',93), ('JCK-ALP-001','2024-06-10',95),
('JCK-ALP-001','2024-06-17',73), ('JCK-ALP-001','2024-06-24',69), ('JCK-ALP-001','2024-07-01',65), ('JCK-ALP-001','2024-07-08',84),
('JCK-ALP-001','2024-07-15',77), ('JCK-ALP-001','2024-07-22',76), ('JCK-ALP-001','2024-07-29',78), ('JCK-ALP-001','2024-08-05',74),
('JCK-ALP-001','2024-08-12',76), ('JCK-ALP-001','2024-08-19',92), ('JCK-ALP-001','2024-08-26',86), ('JCK-ALP-001','2024-09-02',100),
('JCK-ALP-001','2024-09-09',105), ('JCK-ALP-001','2024-09-16',126), ('JCK-ALP-001','2024-09-23',120), ('JCK-ALP-001','2024-09-30',107),
('JCK-ALP-001','2024-10-07',138), ('JCK-ALP-001','2024-10-14',142), ('JCK-ALP-001','2024-10-21',153), ('JCK-ALP-001','2024-10-28',150),
('JCK-ALP-001','2024-11-04',164), ('JCK-ALP-001','2024-11-11',162), ('JCK-ALP-001','2024-11-18',165), ('JCK-ALP-001','2024-11-25',187),
('JCK-ALP-001','2024-12-02',176), ('JCK-ALP-001','2024-12-09',193), ('JCK-ALP-001','2024-12-16',207), ('JCK-ALP-001','2024-12-23',197);

-- JCK-STM-002 (Summit Down Jacket) — 104 weeks
INSERT INTO demand_history (sku_id, week_start, units_sold) VALUES
('JCK-STM-002','2023-01-02',150), ('JCK-STM-002','2023-01-09',147), ('JCK-STM-002','2023-01-16',134), ('JCK-STM-002','2023-01-23',145),
('JCK-STM-002','2023-01-30',139), ('JCK-STM-002','2023-02-06',130), ('JCK-STM-002','2023-02-13',136), ('JCK-STM-002','2023-02-20',111),
('JCK-STM-002','2023-02-27',118), ('JCK-STM-002','2023-03-06',103), ('JCK-STM-002','2023-03-13',92), ('JCK-STM-002','2023-03-20',79),
('JCK-STM-002','2023-03-27',65), ('JCK-STM-002','2023-04-03',62), ('JCK-STM-002','2023-04-10',59), ('JCK-STM-002','2023-04-17',53),
('JCK-STM-002','2023-04-24',73), ('JCK-STM-002','2023-05-01',45), ('JCK-STM-002','2023-05-08',40), ('JCK-STM-002','2023-05-15',38),
('JCK-STM-002','2023-05-22',28), ('JCK-STM-002','2023-05-29',26), ('JCK-STM-002','2023-06-05',24), ('JCK-STM-002','2023-06-12',9),
('JCK-STM-002','2023-06-19',19), ('JCK-STM-002','2023-06-26',40), ('JCK-STM-002','2023-07-03',16), ('JCK-STM-002','2023-07-10',36),
('JCK-STM-002','2023-07-17',40), ('JCK-STM-002','2023-07-24',47), ('JCK-STM-002','2023-07-31',70), ('JCK-STM-002','2023-08-07',71),
('JCK-STM-002','2023-08-14',73), ('JCK-STM-002','2023-08-21',73), ('JCK-STM-002','2023-08-28',81), ('JCK-STM-002','2023-09-04',83),
('JCK-STM-002','2023-09-11',91), ('JCK-STM-002','2023-09-18',102), ('JCK-STM-002','2023-09-25',96), ('JCK-STM-002','2023-10-02',116),
('JCK-STM-002','2023-10-09',149), ('JCK-STM-002','2023-10-16',123), ('JCK-STM-002','2023-10-23',152), ('JCK-STM-002','2023-10-30',143),
('JCK-STM-002','2023-11-06',164), ('JCK-STM-002','2023-11-13',181), ('JCK-STM-002','2023-11-20',247), ('JCK-STM-002','2023-11-27',165),
('JCK-STM-002','2023-12-04',169), ('JCK-STM-002','2023-12-11',174), ('JCK-STM-002','2023-12-18',259), ('JCK-STM-002','2023-12-25',147),
('JCK-STM-002','2024-01-01',183), ('JCK-STM-002','2024-01-08',166), ('JCK-STM-002','2024-01-15',178), ('JCK-STM-002','2024-01-22',143),
('JCK-STM-002','2024-01-29',150), ('JCK-STM-002','2024-02-05',139), ('JCK-STM-002','2024-02-12',129), ('JCK-STM-002','2024-02-19',126),
('JCK-STM-002','2024-02-26',114), ('JCK-STM-002','2024-03-04',109), ('JCK-STM-002','2024-03-11',109), ('JCK-STM-002','2024-03-18',110),
('JCK-STM-002','2024-03-25',93), ('JCK-STM-002','2024-04-01',85), ('JCK-STM-002','2024-04-08',62), ('JCK-STM-002','2024-04-15',56),
('JCK-STM-002','2024-04-22',63), ('JCK-STM-002','2024-04-29',57), ('JCK-STM-002','2024-05-06',46), ('JCK-STM-002','2024-05-13',48),
('JCK-STM-002','2024-05-20',50), ('JCK-STM-002','2024-05-27',37), ('JCK-STM-002','2024-06-03',50), ('JCK-STM-002','2024-06-10',40),
('JCK-STM-002','2024-06-17',29), ('JCK-STM-002','2024-06-24',39), ('JCK-STM-002','2024-07-01',41), ('JCK-STM-002','2024-07-08',46),
('JCK-STM-002','2024-07-15',60), ('JCK-STM-002','2024-07-22',69), ('JCK-STM-002','2024-07-29',75), ('JCK-STM-002','2024-08-05',73),
('JCK-STM-002','2024-08-12',80), ('JCK-STM-002','2024-08-19',103), ('JCK-STM-002','2024-08-26',81), ('JCK-STM-002','2024-09-02',101),
('JCK-STM-002','2024-09-09',109), ('JCK-STM-002','2024-09-16',132), ('JCK-STM-002','2024-09-23',118), ('JCK-STM-002','2024-09-30',140),
('JCK-STM-002','2024-10-07',136), ('JCK-STM-002','2024-10-14',146), ('JCK-STM-002','2024-10-21',160), ('JCK-STM-002','2024-10-28',156),
('JCK-STM-002','2024-11-04',172), ('JCK-STM-002','2024-11-11',179), ('JCK-STM-002','2024-11-18',264), ('JCK-STM-002','2024-11-25',183),
('JCK-STM-002','2024-12-02',198), ('JCK-STM-002','2024-12-09',193), ('JCK-STM-002','2024-12-16',263), ('JCK-STM-002','2024-12-23',177);

-- ACC-BEA-010 (Merino Beanie) — 104 weeks
INSERT INTO demand_history (sku_id, week_start, units_sold) VALUES
('ACC-BEA-010','2023-01-02',57), ('ACC-BEA-010','2023-01-09',52), ('ACC-BEA-010','2023-01-16',49), ('ACC-BEA-010','2023-01-23',59),
('ACC-BEA-010','2023-01-30',56), ('ACC-BEA-010','2023-02-06',58), ('ACC-BEA-010','2023-02-13',50), ('ACC-BEA-010','2023-02-20',59),
('ACC-BEA-010','2023-02-27',52), ('ACC-BEA-010','2023-03-06',53), ('ACC-BEA-010','2023-03-13',45), ('ACC-BEA-010','2023-03-20',38),
('ACC-BEA-010','2023-03-27',49), ('ACC-BEA-010','2023-04-03',40), ('ACC-BEA-010','2023-04-10',37), ('ACC-BEA-010','2023-04-17',37),
('ACC-BEA-010','2023-04-24',33), ('ACC-BEA-010','2023-05-01',42), ('ACC-BEA-010','2023-05-08',38), ('ACC-BEA-010','2023-05-15',30),
('ACC-BEA-010','2023-05-22',36), ('ACC-BEA-010','2023-05-29',33), ('ACC-BEA-010','2023-06-05',31), ('ACC-BEA-010','2023-06-12',29),
('ACC-BEA-010','2023-06-19',33), ('ACC-BEA-010','2023-06-26',28), ('ACC-BEA-010','2023-07-03',25), ('ACC-BEA-010','2023-07-10',22),
('ACC-BEA-010','2023-07-17',24), ('ACC-BEA-010','2023-07-24',21), ('ACC-BEA-010','2023-07-31',20), ('ACC-BEA-010','2023-08-07',29),
('ACC-BEA-010','2023-08-14',21), ('ACC-BEA-010','2023-08-21',29), ('ACC-BEA-010','2023-08-28',27), ('ACC-BEA-010','2023-09-04',37),
('ACC-BEA-010','2023-09-11',36), ('ACC-BEA-010','2023-09-18',32), ('ACC-BEA-010','2023-09-25',33), ('ACC-BEA-010','2023-10-02',39),
('ACC-BEA-010','2023-10-09',38), ('ACC-BEA-010','2023-10-16',45), ('ACC-BEA-010','2023-10-23',45), ('ACC-BEA-010','2023-10-30',40),
('ACC-BEA-010','2023-11-06',56), ('ACC-BEA-010','2023-11-13',51), ('ACC-BEA-010','2023-11-20',47), ('ACC-BEA-010','2023-11-27',52),
('ACC-BEA-010','2023-12-04',50), ('ACC-BEA-010','2023-12-11',52), ('ACC-BEA-010','2023-12-18',46), ('ACC-BEA-010','2023-12-25',52),
('ACC-BEA-010','2024-01-01',52), ('ACC-BEA-010','2024-01-08',58), ('ACC-BEA-010','2024-01-15',53), ('ACC-BEA-010','2024-01-22',56),
('ACC-BEA-010','2024-01-29',52), ('ACC-BEA-010','2024-02-05',49), ('ACC-BEA-010','2024-02-12',55), ('ACC-BEA-010','2024-02-19',61),
('ACC-BEA-010','2024-02-26',52), ('ACC-BEA-010','2024-03-04',46), ('ACC-BEA-010','2024-03-11',45), ('ACC-BEA-010','2024-03-18',43),
('ACC-BEA-010','2024-03-25',43), ('ACC-BEA-010','2024-04-01',47), ('ACC-BEA-010','2024-04-08',41), ('ACC-BEA-010','2024-04-15',39),
('ACC-BEA-010','2024-04-22',38), ('ACC-BEA-010','2024-04-29',30), ('ACC-BEA-010','2024-05-06',31), ('ACC-BEA-010','2024-05-13',34),
('ACC-BEA-010','2024-05-20',36), ('ACC-BEA-010','2024-05-27',22), ('ACC-BEA-010','2024-06-03',37), ('ACC-BEA-010','2024-06-10',33),
('ACC-BEA-010','2024-06-17',26), ('ACC-BEA-010','2024-06-24',27), ('ACC-BEA-010','2024-07-01',19), ('ACC-BEA-010','2024-07-08',29),
('ACC-BEA-010','2024-07-15',29), ('ACC-BEA-010','2024-07-22',26), ('ACC-BEA-010','2024-07-29',26), ('ACC-BEA-010','2024-08-05',32),
('ACC-BEA-010','2024-08-12',29), ('ACC-BEA-010','2024-08-19',37), ('ACC-BEA-010','2024-08-26',33), ('ACC-BEA-010','2024-09-02',23),
('ACC-BEA-010','2024-09-09',39), ('ACC-BEA-010','2024-09-16',37), ('ACC-BEA-010','2024-09-23',40), ('ACC-BEA-010','2024-09-30',34),
('ACC-BEA-010','2024-10-07',31), ('ACC-BEA-010','2024-10-14',40), ('ACC-BEA-010','2024-10-21',43), ('ACC-BEA-010','2024-10-28',42),
('ACC-BEA-010','2024-11-04',53), ('ACC-BEA-010','2024-11-11',57), ('ACC-BEA-010','2024-11-18',48), ('ACC-BEA-010','2024-11-25',57),
('ACC-BEA-010','2024-12-02',50), ('ACC-BEA-010','2024-12-09',49), ('ACC-BEA-010','2024-12-16',45), ('ACC-BEA-010','2024-12-23',51);

-- BAG-DAY-020 (Daypack 22L) — 104 weeks
INSERT INTO demand_history (sku_id, week_start, units_sold) VALUES
('BAG-DAY-020','2023-01-02',61), ('BAG-DAY-020','2023-01-09',66), ('BAG-DAY-020','2023-01-16',71), ('BAG-DAY-020','2023-01-23',83),
('BAG-DAY-020','2023-01-30',76), ('BAG-DAY-020','2023-02-06',87), ('BAG-DAY-020','2023-02-13',71), ('BAG-DAY-020','2023-02-20',53),
('BAG-DAY-020','2023-02-27',52), ('BAG-DAY-020','2023-03-06',61), ('BAG-DAY-020','2023-03-13',74), ('BAG-DAY-020','2023-03-20',92),
('BAG-DAY-020','2023-03-27',81), ('BAG-DAY-020','2023-04-03',62), ('BAG-DAY-020','2023-04-10',77), ('BAG-DAY-020','2023-04-17',73),
('BAG-DAY-020','2023-04-24',73), ('BAG-DAY-020','2023-05-01',82), ('BAG-DAY-020','2023-05-08',55), ('BAG-DAY-020','2023-05-15',71),
('BAG-DAY-020','2023-05-22',62), ('BAG-DAY-020','2023-05-29',68), ('BAG-DAY-020','2023-06-05',74), ('BAG-DAY-020','2023-06-12',84),
('BAG-DAY-020','2023-06-19',60), ('BAG-DAY-020','2023-06-26',71), ('BAG-DAY-020','2023-07-03',68), ('BAG-DAY-020','2023-07-10',71),
('BAG-DAY-020','2023-07-17',76), ('BAG-DAY-020','2023-07-24',75), ('BAG-DAY-020','2023-07-31',83), ('BAG-DAY-020','2023-08-07',68),
('BAG-DAY-020','2023-08-14',69), ('BAG-DAY-020','2023-08-21',81), ('BAG-DAY-020','2023-08-28',70), ('BAG-DAY-020','2023-09-04',76),
('BAG-DAY-020','2023-09-11',83), ('BAG-DAY-020','2023-09-18',72), ('BAG-DAY-020','2023-09-25',85), ('BAG-DAY-020','2023-10-02',66),
('BAG-DAY-020','2023-10-09',81), ('BAG-DAY-020','2023-10-16',76), ('BAG-DAY-020','2023-10-23',62), ('BAG-DAY-020','2023-10-30',80),
('BAG-DAY-020','2023-11-06',78), ('BAG-DAY-020','2023-11-13',64), ('BAG-DAY-020','2023-11-20',55), ('BAG-DAY-020','2023-11-27',78),
('BAG-DAY-020','2023-12-04',66), ('BAG-DAY-020','2023-12-11',66), ('BAG-DAY-020','2023-12-18',96), ('BAG-DAY-020','2023-12-25',73),
('BAG-DAY-020','2024-01-01',59), ('BAG-DAY-020','2024-01-08',75), ('BAG-DAY-020','2024-01-15',74), ('BAG-DAY-020','2024-01-22',68),
('BAG-DAY-020','2024-01-29',83), ('BAG-DAY-020','2024-02-05',80), ('BAG-DAY-020','2024-02-12',80), ('BAG-DAY-020','2024-02-19',65),
('BAG-DAY-020','2024-02-26',78), ('BAG-DAY-020','2024-03-04',79), ('BAG-DAY-020','2024-03-11',77), ('BAG-DAY-020','2024-03-18',77),
('BAG-DAY-020','2024-03-25',65), ('BAG-DAY-020','2024-04-01',78), ('BAG-DAY-020','2024-04-08',59), ('BAG-DAY-020','2024-04-15',81),
('BAG-DAY-020','2024-04-22',64), ('BAG-DAY-020','2024-04-29',75), ('BAG-DAY-020','2024-05-06',74), ('BAG-DAY-020','2024-05-13',78),
('BAG-DAY-020','2024-05-20',68), ('BAG-DAY-020','2024-05-27',85), ('BAG-DAY-020','2024-06-03',64), ('BAG-DAY-020','2024-06-10',79),
('BAG-DAY-020','2024-06-17',76), ('BAG-DAY-020','2024-06-24',69), ('BAG-DAY-020','2024-07-01',69), ('BAG-DAY-020','2024-07-08',56),
('BAG-DAY-020','2024-07-15',75), ('BAG-DAY-020','2024-07-22',84), ('BAG-DAY-020','2024-07-29',67), ('BAG-DAY-020','2024-08-05',80),
('BAG-DAY-020','2024-08-12',73), ('BAG-DAY-020','2024-08-19',62), ('BAG-DAY-020','2024-08-26',74), ('BAG-DAY-020','2024-09-02',79),
('BAG-DAY-020','2024-09-09',69), ('BAG-DAY-020','2024-09-16',69), ('BAG-DAY-020','2024-09-23',71), ('BAG-DAY-020','2024-09-30',67),
('BAG-DAY-020','2024-10-07',66), ('BAG-DAY-020','2024-10-14',68), ('BAG-DAY-020','2024-10-21',68), ('BAG-DAY-020','2024-10-28',73),
('BAG-DAY-020','2024-11-04',71), ('BAG-DAY-020','2024-11-11',70), ('BAG-DAY-020','2024-11-18',71), ('BAG-DAY-020','2024-11-25',67),
('BAG-DAY-020','2024-12-02',79), ('BAG-DAY-020','2024-12-09',86), ('BAG-DAY-020','2024-12-16',66), ('BAG-DAY-020','2024-12-23',87);

-- FLC-ZIP-030 (Fleece Half-Zip) — 104 weeks
INSERT INTO demand_history (sku_id, week_start, units_sold) VALUES
('FLC-ZIP-030','2023-01-02',39), ('FLC-ZIP-030','2023-01-09',56), ('FLC-ZIP-030','2023-01-16',55), ('FLC-ZIP-030','2023-01-23',75),
('FLC-ZIP-030','2023-01-30',66), ('FLC-ZIP-030','2023-02-06',60), ('FLC-ZIP-030','2023-02-13',63), ('FLC-ZIP-030','2023-02-20',80),
('FLC-ZIP-030','2023-02-27',111), ('FLC-ZIP-030','2023-03-06',72), ('FLC-ZIP-030','2023-03-13',75), ('FLC-ZIP-030','2023-03-20',82),
('FLC-ZIP-030','2023-03-27',85), ('FLC-ZIP-030','2023-04-03',72), ('FLC-ZIP-030','2023-04-10',77), ('FLC-ZIP-030','2023-04-17',78),
('FLC-ZIP-030','2023-04-24',76), ('FLC-ZIP-030','2023-05-01',92), ('FLC-ZIP-030','2023-05-08',86), ('FLC-ZIP-030','2023-05-15',82),
('FLC-ZIP-030','2023-05-22',67), ('FLC-ZIP-030','2023-05-29',87), ('FLC-ZIP-030','2023-06-05',88), ('FLC-ZIP-030','2023-06-12',85),
('FLC-ZIP-030','2023-06-19',82), ('FLC-ZIP-030','2023-06-26',76), ('FLC-ZIP-030','2023-07-03',81), ('FLC-ZIP-030','2023-07-10',77),
('FLC-ZIP-030','2023-07-17',76), ('FLC-ZIP-030','2023-07-24',69), ('FLC-ZIP-030','2023-07-31',74), ('FLC-ZIP-030','2023-08-07',69),
('FLC-ZIP-030','2023-08-14',73), ('FLC-ZIP-030','2023-08-21',62), ('FLC-ZIP-030','2023-08-28',62), ('FLC-ZIP-030','2023-09-04',43),
('FLC-ZIP-030','2023-09-11',50), ('FLC-ZIP-030','2023-09-18',51), ('FLC-ZIP-030','2023-09-25',52), ('FLC-ZIP-030','2023-10-02',53),
('FLC-ZIP-030','2023-10-09',58), ('FLC-ZIP-030','2023-10-16',49), ('FLC-ZIP-030','2023-10-23',43), ('FLC-ZIP-030','2023-10-30',54),
('FLC-ZIP-030','2023-11-06',63), ('FLC-ZIP-030','2023-11-13',53), ('FLC-ZIP-030','2023-11-20',66), ('FLC-ZIP-030','2023-11-27',67),
('FLC-ZIP-030','2023-12-04',72), ('FLC-ZIP-030','2023-12-11',75), ('FLC-ZIP-030','2023-12-18',75), ('FLC-ZIP-030','2023-12-25',78),
('FLC-ZIP-030','2024-01-01',86), ('FLC-ZIP-030','2024-01-08',61), ('FLC-ZIP-030','2024-01-15',76), ('FLC-ZIP-030','2024-01-22',95),
('FLC-ZIP-030','2024-01-29',84), ('FLC-ZIP-030','2024-02-05',89), ('FLC-ZIP-030','2024-02-12',104), ('FLC-ZIP-030','2024-02-19',103),
('FLC-ZIP-030','2024-02-26',126), ('FLC-ZIP-030','2024-03-04',99), ('FLC-ZIP-030','2024-03-11',102), ('FLC-ZIP-030','2024-03-18',100),
('FLC-ZIP-030','2024-03-25',108), ('FLC-ZIP-030','2024-04-01',100), ('FLC-ZIP-030','2024-04-08',98), ('FLC-ZIP-030','2024-04-15',111),
('FLC-ZIP-030','2024-04-22',107), ('FLC-ZIP-030','2024-04-29',118), ('FLC-ZIP-030','2024-05-06',108), ('FLC-ZIP-030','2024-05-13',108),
('FLC-ZIP-030','2024-05-20',113), ('FLC-ZIP-030','2024-05-27',109), ('FLC-ZIP-030','2024-06-03',100), ('FLC-ZIP-030','2024-06-10',94),
('FLC-ZIP-030','2024-06-17',97), ('FLC-ZIP-030','2024-06-24',96), ('FLC-ZIP-030','2024-07-01',110), ('FLC-ZIP-030','2024-07-08',99),
('FLC-ZIP-030','2024-07-15',85), ('FLC-ZIP-030','2024-07-22',94), ('FLC-ZIP-030','2024-07-29',94), ('FLC-ZIP-030','2024-08-05',90),
('FLC-ZIP-030','2024-08-12',96), ('FLC-ZIP-030','2024-08-19',80), ('FLC-ZIP-030','2024-08-26',103), ('FLC-ZIP-030','2024-09-02',85),
('FLC-ZIP-030','2024-09-09',84), ('FLC-ZIP-030','2024-09-16',80), ('FLC-ZIP-030','2024-09-23',79), ('FLC-ZIP-030','2024-09-30',76),
('FLC-ZIP-030','2024-10-07',66), ('FLC-ZIP-030','2024-10-14',80), ('FLC-ZIP-030','2024-10-21',77), ('FLC-ZIP-030','2024-10-28',77),
('FLC-ZIP-030','2024-11-04',82), ('FLC-ZIP-030','2024-11-11',87), ('FLC-ZIP-030','2024-11-18',95), ('FLC-ZIP-030','2024-11-25',91),
('FLC-ZIP-030','2024-12-02',86), ('FLC-ZIP-030','2024-12-09',87), ('FLC-ZIP-030','2024-12-16',105), ('FLC-ZIP-030','2024-12-23',112);

-- SAN-TRL-040 (Trail Sandal) — 104 weeks
INSERT INTO demand_history (sku_id, week_start, units_sold) VALUES
('SAN-TRL-040','2023-01-02',18), ('SAN-TRL-040','2023-01-09',19), ('SAN-TRL-040','2023-01-16',28), ('SAN-TRL-040','2023-01-23',20),
('SAN-TRL-040','2023-01-30',26), ('SAN-TRL-040','2023-02-06',29), ('SAN-TRL-040','2023-02-13',26), ('SAN-TRL-040','2023-02-20',40),
('SAN-TRL-040','2023-02-27',27), ('SAN-TRL-040','2023-03-06',30), ('SAN-TRL-040','2023-03-13',50), ('SAN-TRL-040','2023-03-20',63),
('SAN-TRL-040','2023-03-27',51), ('SAN-TRL-040','2023-04-03',68), ('SAN-TRL-040','2023-04-10',77), ('SAN-TRL-040','2023-04-17',82),
('SAN-TRL-040','2023-04-24',84), ('SAN-TRL-040','2023-05-01',73), ('SAN-TRL-040','2023-05-08',86), ('SAN-TRL-040','2023-05-15',80),
('SAN-TRL-040','2023-05-22',84), ('SAN-TRL-040','2023-05-29',96), ('SAN-TRL-040','2023-06-05',105), ('SAN-TRL-040','2023-06-12',107),
('SAN-TRL-040','2023-06-19',77), ('SAN-TRL-040','2023-06-26',89), ('SAN-TRL-040','2023-07-03',119), ('SAN-TRL-040','2023-07-10',103),
('SAN-TRL-040','2023-07-17',87), ('SAN-TRL-040','2023-07-24',115), ('SAN-TRL-040','2023-07-31',88), ('SAN-TRL-040','2023-08-07',88),
('SAN-TRL-040','2023-08-14',87), ('SAN-TRL-040','2023-08-21',77), ('SAN-TRL-040','2023-08-28',82), ('SAN-TRL-040','2023-09-04',79),
('SAN-TRL-040','2023-09-11',66), ('SAN-TRL-040','2023-09-18',60), ('SAN-TRL-040','2023-09-25',59), ('SAN-TRL-040','2023-10-02',42),
('SAN-TRL-040','2023-10-09',41), ('SAN-TRL-040','2023-10-16',46), ('SAN-TRL-040','2023-10-23',18), ('SAN-TRL-040','2023-10-30',50),
('SAN-TRL-040','2023-11-06',46), ('SAN-TRL-040','2023-11-13',36), ('SAN-TRL-040','2023-11-20',14), ('SAN-TRL-040','2023-11-27',17),
('SAN-TRL-040','2023-12-04',31), ('SAN-TRL-040','2023-12-11',12), ('SAN-TRL-040','2023-12-18',14), ('SAN-TRL-040','2023-12-25',0),
('SAN-TRL-040','2024-01-01',13), ('SAN-TRL-040','2024-01-08',0), ('SAN-TRL-040','2024-01-15',8), ('SAN-TRL-040','2024-01-22',10),
('SAN-TRL-040','2024-01-29',27), ('SAN-TRL-040','2024-02-05',19), ('SAN-TRL-040','2024-02-12',23), ('SAN-TRL-040','2024-02-19',22),
('SAN-TRL-040','2024-02-26',18), ('SAN-TRL-040','2024-03-04',36), ('SAN-TRL-040','2024-03-11',22), ('SAN-TRL-040','2024-03-18',40),
('SAN-TRL-040','2024-03-25',35), ('SAN-TRL-040','2024-04-01',47), ('SAN-TRL-040','2024-04-08',76), ('SAN-TRL-040','2024-04-15',69),
('SAN-TRL-040','2024-04-22',64), ('SAN-TRL-040','2024-04-29',61), ('SAN-TRL-040','2024-05-06',82), ('SAN-TRL-040','2024-05-13',73),
('SAN-TRL-040','2024-05-20',81), ('SAN-TRL-040','2024-05-27',95), ('SAN-TRL-040','2024-06-03',88), ('SAN-TRL-040','2024-06-10',94),
('SAN-TRL-040','2024-06-17',98), ('SAN-TRL-040','2024-06-24',83), ('SAN-TRL-040','2024-07-01',92), ('SAN-TRL-040','2024-07-08',101),
('SAN-TRL-040','2024-07-15',97), ('SAN-TRL-040','2024-07-22',84), ('SAN-TRL-040','2024-07-29',93), ('SAN-TRL-040','2024-08-05',67),
('SAN-TRL-040','2024-08-12',74), ('SAN-TRL-040','2024-08-19',91), ('SAN-TRL-040','2024-08-26',84), ('SAN-TRL-040','2024-09-02',54),
('SAN-TRL-040','2024-09-09',68), ('SAN-TRL-040','2024-09-16',61), ('SAN-TRL-040','2024-09-23',52), ('SAN-TRL-040','2024-09-30',45),
('SAN-TRL-040','2024-10-07',48), ('SAN-TRL-040','2024-10-14',45), ('SAN-TRL-040','2024-10-21',17), ('SAN-TRL-040','2024-10-28',36),
('SAN-TRL-040','2024-11-04',24), ('SAN-TRL-040','2024-11-11',19), ('SAN-TRL-040','2024-11-18',0), ('SAN-TRL-040','2024-11-25',3),
('SAN-TRL-040','2024-12-02',9), ('SAN-TRL-040','2024-12-09',14), ('SAN-TRL-040','2024-12-16',0), ('SAN-TRL-040','2024-12-23',1);
```

Sanity check — this should print `624`:

```sql
SELECT COUNT(*) FROM demand_history;
```

And this should print 6 rows, one per SKU, with very different shapes (run it now, before any lecture — look at the `min`/`max`/`avg` spread and guess which SKU is which pattern before you read Lecture 1):

```sql
SELECT sku_id, MIN(units_sold), MAX(units_sold), ROUND(AVG(units_sold), 1) AS avg_units
FROM demand_history
GROUP BY sku_id
ORDER BY sku_id;
```

Keep this data exactly as loaded — every lecture, exercise, and challenge this week references specific weeks and specific numbers computed from it.

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-demand-patterns-and-decomposition.md](./lecture-notes/01-demand-patterns-and-decomposition.md) | Level, trend, seasonality, noise; additive vs. multiplicative decomposition; a worked toy example plus the six real SKUs | 2h |
| 2 | [lecture-notes/02-baseline-and-moving-average-methods.md](./lecture-notes/02-baseline-and-moving-average-methods.md) | Naive, seasonal naive, moving average — built from SQL, scored by hand | 2h |
| 3 | [lecture-notes/03-exponential-smoothing-and-error-metrics.md](./lecture-notes/03-exponential-smoothing-and-error-metrics.md) | Single & double exponential smoothing (Holt); MAE, MAPE, RMSE, bias | 2h |
| 4 | [exercises/exercise-01-pull-demand-from-sql-to-pandas.md](./exercises/exercise-01-pull-demand-from-sql-to-pandas.md) | Query `demand_history`, load it into pandas, reshape it for forecasting | 1h |
| 5 | [exercises/exercise-02-moving-average-forecast.md](./exercises/exercise-02-moving-average-forecast.md) | Build naive, seasonal naive, and moving-average forecasts in pandas | 1.5h |
| 6 | [exercises/exercise-03-score-forecast-error.md](./exercises/exercise-03-score-forecast-error.md) | Compute MAE/MAPE/RMSE/bias for every method and rank them | 1h |
| 7 | [challenges/challenge-01-choose-a-baseline-per-sku.md](./challenges/challenge-01-choose-a-baseline-per-sku.md) | Pick (and defend) the right baseline for each of the six SKU shapes | 1h |
| 8 | [challenges/challenge-02-seasonal-naive-vs-smoothing.md](./challenges/challenge-02-seasonal-naive-vs-smoothing.md) | Run a face-off: seasonal naive vs. exponential smoothing, tune α/β, explain the winner | 1h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Forecast 50 SKUs from SQL history; score MAPE/bias vs. naive for each | 3h |
| 10 | [homework.md](./homework.md) | Extra practice, spread across the week | 4.5h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official/free references + tools to install | — |

## Weekly schedule

Adds up to roughly the course's **~28 hr/week full-time pace**. Adjust to your own pace per the syllabus.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|-------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Seed load; decomposition | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | Baselines & moving average | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Wednesday | Exponential smoothing | 2h | 0h | 1h | 0.5h | 1h | 0.5h | 5h |
| Thursday | Error metrics; scoring | 0h | 1h | 1h | 0.5h | 1h | 1h | 4.5h |
| Friday | Challenges; per-SKU judgment | 0h | 0h | 1h | 0.5h | 0.5h | 1h | 3h |
| Saturday | Mini-project (50-SKU report) | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **3.5h** | **3h** | **3.5h** | **4.5h** | **5h** | **28.5h** |

## By the end of this week you can…

- Look at any raw demand series and describe its level, trend, seasonal shape, and noise level before touching a formula.
- Build naive, seasonal naive, moving-average, single-smoothing, and double-smoothing forecasts entirely in SQL + pandas.
- Score any forecast on MAE, MAPE, RMSE, and bias, and explain in one sentence what each metric would tell a non-technical planner.
- Refuse to ship a forecasting method — no matter how sophisticated — until you've shown it beats a naive benchmark on held-out weeks.

## Up next

Week 4 — Advanced forecasting: ML methods, hierarchical reconciliation, and proper backtesting, building directly on the baselines and error metrics you now have memorized.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
