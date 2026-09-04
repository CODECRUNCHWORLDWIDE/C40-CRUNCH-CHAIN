# Week 8 — Warehousing & Fulfillment

> **Goal:** by Sunday you can take a real order-line pick profile from a distribution center, run an ABC velocity analysis on it, re-slot the warehouse to cut pick travel, and size a batching/labor plan to hit a throughput target — every computation in SQL and Python, never a spreadsheet.

Welcome to **C40 · Crunch Chain**, Week 8. Week 7 got freight to the door of **Austin East DC**. This week we go inside the four walls and optimize how it moves from receiving dock to delivery truck. Every distribution center runs the same flow — receive, putaway, store, pick, pack, ship — and in almost every DC the single biggest lever an analyst can pull without buying new racking or hiring more people is **where things are stored relative to how often they're picked**. Put a fast-moving SKU forty feet from the pack station because that's where it happened to fit on move-in day, and you pay for that mistake on every single pick, forever, until someone runs the numbers and fixes it. That someone is you, this week.

We work against a real one-month **order-line pick profile** from Austin East DC: **225 pick lines** across **76 customer orders**, pulled from **20 SKUs** in the outdoor-gear catalog. You'll classify those SKUs by pick velocity (ABC analysis), discover the warehouse's current slotting is almost the *opposite* of what the data says it should be, quantify exactly how many feet of walking that mistake costs every month, fix it, and then build a batching and labor plan sized to a real throughput target.

**Data rule for this course:** every table, every velocity ranking, every slotting and labor decision this week is done in **SQL (PostgreSQL 16, SQLite fallback)** and/or **Python (pandas)** — never a spreadsheet. A pick-frequency report is a `GROUP BY` and a `SUM`, not a hand-sorted worksheet; treating warehouse data as *data* is what lets you re-slot the whole DC in ten seconds when next quarter's sales mix shifts, instead of re-doing a manual worksheet from scratch.

## Learning objectives

By the end of this week, you will be able to:

- **Trace** the receive-to-ship flow inside a distribution center and identify where labor and time cost concentrate at each step.
- **Run** an ABC (Pareto) velocity analysis on an order-line pick profile in SQL, and classify SKUs into A/B/C tiers from cumulative pick-frequency share.
- **Design** a slotting plan that puts high-velocity SKUs in golden-zone locations, and **compute** the pick-travel distance a slotting plan produces.
- **Batch** orders into waves that share travel across a pick, and **quantify** how much batching reduces total trips versus one-order-at-a-time picking.
- **Size** a picking labor plan to a stated throughput target, and **measure** fulfillment performance with the three metrics that matter: throughput, order cycle time, and pick productivity.

## Standards this week meets

| Bar | What this week is measured against |
| --- | --- |
| University | `ISM 4400` — describe the receive-to-ship flow inside a distribution centre, and design storage, picking and labour to a stated throughput target. |
| Industry | Re-slot a distribution centre and report, in feet of pick travel and in labour hours, exactly what the change saves and what it costs to make. |
| Beyond the bar | Order batching is layered on top of the re-slot and its trip reduction is measured on the real pick profile rather than asserted, so the two levers can be compared against each other — `exercises/exercise-03-order-batching-heuristic.md` |

## Prerequisites

- Comfortable with `SELECT`, `WHERE`, `GROUP BY`, `HAVING`, joins, and window functions in SQL (Weeks 2–7 of this course, or [C33 Crunch SQL](../../../C33-CRUNCH-SQL/)).
- Comfortable reading and writing basic pandas: `groupby`, `sort_values`, `cumsum`, and simple loops — the ABC analysis and slotting exercises lean on `cumsum()` heavily.
- Basic arithmetic with rates and units (distance ÷ speed = time; this week has no calculus, unlike Weeks 3–5).
- PostgreSQL 16+ **or** SQLite 3.35+, plus Python 3.10+ with `pandas` and `numpy`. See [`resources.md`](./resources.md) for install steps.

## Setup — seed the Week 8 warehouse tables

Everything this week runs against three tables: the SKU catalog, the pick-line history, and the warehouse's storage slots. Create all three once, before Lecture 1.

**PostgreSQL:**

```bash
createdb crunch_chain_wk8
psql crunch_chain_wk8
```

**SQLite:**

```bash
sqlite3 crunch_chain_wk8.db
```

### Table 1 — `warehouse_skus`

The 20 SKUs that move through Austin East DC's fulfillment operation this month. `unit_cube_ft` and `units_per_case` matter for slotting — a slot has to physically hold the item and a sane amount of buffer stock, not just be "close."

```sql
CREATE TABLE warehouse_skus (
    sku_id         INTEGER PRIMARY KEY,
    sku_name       TEXT    NOT NULL,
    category       TEXT    NOT NULL,
    unit_cube_ft   NUMERIC NOT NULL,   -- cubic feet per single unit
    units_per_case INTEGER NOT NULL,   -- units in one replenishment case
    unit_cost      NUMERIC NOT NULL
);

INSERT INTO warehouse_skus VALUES
(1,  'Hydration Bladder 2L',          'Hydration',       0.05, 24,  9.50),
(2,  'Trail Wool Socks 3-Pack',       'Apparel',         0.03, 48, 12.00),
(3,  'Headlamp 300-Lumen',            'Electronics',     0.04, 36, 18.00),
(4,  'Rain Shell Poncho',             'Apparel',         0.06, 40, 14.00),
(5,  'Insulated Water Bottle 32oz',   'Hydration',       0.08, 24, 16.00),
(6,  'Trekking Pole Pair',            'Gear',            0.35, 12, 42.00),
(7,  'Camp Mug Enamel',               'Camp Kitchen',    0.05, 36,  7.50),
(8,  'First Aid Kit Trail',           'Safety',          0.10, 24, 15.00),
(9,  'Daypack 20L',                   'Packs',           0.60, 10, 45.00),
(10, 'Dry Bag 10L',                   'Gear',            0.15, 20, 11.00),
(11, 'Compression Sack Set',          'Gear',            0.12, 24, 13.50),
(12, 'Trail Runner Shoes Mens',       'Footwear',        0.45, 12, 55.00),
(13, 'Trail Runner Shoes Womens',     'Footwear',        0.42, 12, 55.00),
(14, 'Backpacking Cookset',           'Camp Kitchen',    0.30, 12, 28.00),
(15, 'Camp Chair Ultralight',         'Camp Furniture',  0.55,  8, 38.00),
(16, 'Ultralight Tent 2P',            'Shelter',         1.80,  4, 220.00),
(17, '3-Season Sleeping Bag',         'Sleep System',    1.20,  6, 95.00),
(18, 'Backpacking Stove Kit',         'Camp Kitchen',    0.25, 16, 32.00),
(19, '60L Expedition Backpack',       'Packs',           2.20,  4, 130.00),
(20, '4-Season Mountaineering Tent',  'Shelter',         3.50,  2, 380.00);
```

Sanity check — should print `20`:

```sql
SELECT COUNT(*) FROM warehouse_skus;
```

Notice the pattern once you skim the table: small, cheap, everyday items (`Hydration Bladder`, `Socks`, `Headlamp`) sit at the top; large, expensive, occasional-purchase items (`4-Season Mountaineering Tent`) sit at the bottom. That's not an accident of this seed data — it's how most catalogs actually shake out, and it's exactly why ABC analysis works: cube and cost correlate (loosely) with *how rarely* something gets picked.

### Table 2 — `pick_lines`

One month (20 business days, June 2025) of real order-line picks at Austin East DC — every line a picker walked to a slot and pulled. This is the **order-line pick profile** the whole week is built on.

```sql
CREATE TABLE pick_lines (
    pick_line_id INTEGER PRIMARY KEY,
    order_id     INTEGER NOT NULL,
    order_date   DATE    NOT NULL,
    sku_id       INTEGER NOT NULL REFERENCES warehouse_skus(sku_id),
    qty_picked   INTEGER NOT NULL
);
```

Now load the 225 rows — the full seed is in [`resources.md`](./resources.md#full-pick_lines-seed-data) to keep this page short, but here's a representative slice so you can see the shape:

```sql
INSERT INTO pick_lines VALUES
(1,1,'2025-06-03',3,2),
(2,2,'2025-06-16',11,1),
(3,2,'2025-06-16',1,2),
(11,7,'2025-06-24',3,1),
(12,7,'2025-06-24',2,1),
(13,7,'2025-06-24',1,3),
(151,52,'2025-06-12',3,1),
(152,52,'2025-06-12',9,2),
(224,76,'2025-06-13',1,1),
(225,76,'2025-06-13',6,1);
-- ... 215 more rows — full block in resources.md
```

Sanity check — this should print `225`:

```sql
SELECT COUNT(*) FROM pick_lines;
```

And this should print `76`:

```sql
SELECT COUNT(DISTINCT order_id) FROM pick_lines;
```

An **order line** is one SKU on one order — `order_id 7` above has three lines (SKUs 3, 2, and 1), each a separate walk-and-pick event in the naive picking model we start with in Lecture 1. Orders run 1–5 lines each; most run 2–3. That line-level grain is what makes ABC analysis, pick-path distance, and batching all computable straight from this one table.

### Table 3 — `warehouse_slots`

Austin East DC's storage locations: **6 golden-zone slots** (closest to the pack/ship station), **10 middle-zone slots**, and **4 currently-occupied reserve-zone slots** (4 more reserve slots sit empty, held for growth). `distance_ft` is one-way walking distance from the pack station to that slot. `current_sku_id` is **today's actual assignment** — set up years ago, alphabetically by SKU name, by whoever built the racking and never revisited it.

```sql
CREATE TABLE warehouse_slots (
    slot_id         TEXT PRIMARY KEY,
    zone            TEXT    NOT NULL,   -- 'Golden', 'Middle', 'Reserve'
    distance_ft     NUMERIC NOT NULL,   -- one-way from pack/ship station
    current_sku_id  INTEGER REFERENCES warehouse_skus(sku_id)  -- NULL = empty
);

INSERT INTO warehouse_slots VALUES
('G1',  'Golden',  20, 17),  -- 3-Season Sleeping Bag
('G2',  'Golden',  24, 20),  -- 4-Season Mountaineering Tent
('G3',  'Golden',  28, 19),  -- 60L Expedition Backpack
('G4',  'Golden',  32, 14),  -- Backpacking Cookset
('G5',  'Golden',  36, 18),  -- Backpacking Stove Kit
('G6',  'Golden',  40, 15),  -- Camp Chair Ultralight
('M1',  'Middle',  55,  7),  -- Camp Mug Enamel
('M2',  'Middle',  62, 11),  -- Compression Sack Set
('M3',  'Middle',  68,  9),  -- Daypack 20L
('M4',  'Middle',  74, 10),  -- Dry Bag 10L
('M5',  'Middle',  80,  8),  -- First Aid Kit Trail
('M6',  'Middle',  86,  3),  -- Headlamp 300-Lumen
('M7',  'Middle',  92,  1),  -- Hydration Bladder 2L
('M8',  'Middle',  98,  5),  -- Insulated Water Bottle 32oz
('M9',  'Middle', 104,  4),  -- Rain Shell Poncho
('M10', 'Middle', 110, 12),  -- Trail Runner Shoes Mens
('R1',  'Reserve',130, 13),  -- Trail Runner Shoes Womens
('R2',  'Reserve',145,  2),  -- Trail Wool Socks 3-Pack
('R3',  'Reserve',160,  6),  -- Trekking Pole Pair
('R4',  'Reserve',175, 16),  -- Ultralight Tent 2P
('R5',  'Reserve',190, NULL),
('R6',  'Reserve',205, NULL),
('R7',  'Reserve',220, NULL),
('R8',  'Reserve',235, NULL);
```

Sanity check — should print `24`:

```sql
SELECT COUNT(*) FROM warehouse_slots;
```

Look closely at that current assignment before you write a single query: `Hydration Bladder 2L` (SKU 1) — read the pick_lines table and you'll find it's the single most-picked SKU in the building — sits in slot `M7`, 92 feet out. Meanwhile `4-Season Mountaineering Tent` (SKU 20), picked exactly **once** all month, sits in golden slot `G2`, 24 feet from the pack station. That is not a hypothetical mistake. It is sitting in this seed data right now, and Lecture 2 will make you put a number on exactly how much it costs.

## Weekly schedule

Adds up to roughly the course's **~28 hr/week full-time pace**.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|--------------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Warehouse flows: receive → putaway → store → pick → pack → ship | 2h | 0h | 0h | 0.5h | 1h | 0h | 3.5h |
| Tuesday | ABC velocity analysis in SQL | 0h | 1.5h | 0h | 0.5h | 1h | 0h | 3h |
| Wednesday | Slotting & pick-path distance | 2h | 1.5h | 1h | 0.5h | 1h | 0h | 6h |
| Thursday | Batching, wave picking, labor sizing | 2h | 1h | 1h | 0.5h | 1h | 1h | 6.5h |
| Friday | Fulfillment metrics; catch-up | 0h | 1h | 1h | 0.5h | 1h | 1.5h | 5h |
| Saturday | Mini-project — slot + batch the DC | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **4h** | **5h** | **3h** | **3.5h** | **5h** | **5h** | **27.5h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-warehouse-operations-and-flows.md](./lecture-notes/01-warehouse-operations-and-flows.md) | The receive-to-ship flow, storage strategies, where time and labor cost accumulate | 2h |
| 2 | [lecture-notes/02-slotting-and-pick-path-optimization.md](./lecture-notes/02-slotting-and-pick-path-optimization.md) | ABC velocity analysis, golden-zone slotting, minimizing pick-path travel | 2h |
| 3 | [lecture-notes/03-order-batching-and-labor-planning.md](./lecture-notes/03-order-batching-and-labor-planning.md) | Wave/batch picking, sizing labor to a throughput target, cycle time & productivity | 2h |
| 4 | [exercises/exercise-01-abc-analysis-for-slotting.md](./exercises/exercise-01-abc-analysis-for-slotting.md) | Classify all 20 SKUs into A/B/C tiers from the pick-line profile | 1.5h |
| 5 | [exercises/exercise-02-pick-path-distance.md](./exercises/exercise-02-pick-path-distance.md) | Compute current vs. re-slotted total pick-travel distance | 1.5h |
| 6 | [exercises/exercise-03-order-batching-heuristic.md](./exercises/exercise-03-order-batching-heuristic.md) | Batch a day's orders into waves and measure the trip reduction | 1h |
| 7 | [challenges/challenge-01-slotting-optimization.md](./challenges/challenge-01-slotting-optimization.md) | Re-slot under a replenishment-frequency constraint, not distance alone | 1.5h |
| 8 | [challenges/challenge-02-labor-capacity-planning.md](./challenges/challenge-02-labor-capacity-planning.md) | Size a picking crew to a stated peak-day throughput target | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Slot the DC, design a batching plan, quantify travel and labor savings | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice tying velocity, slotting, batching, and labor together | 5h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Full seed data, official references, tools to install | — |

## By the end of this week you can…

- Draw the six-step receive-to-ship flow from memory and name, for each step, the one metric an operations manager watches.
- Write a single SQL query that ranks every SKU by pick frequency, computes its cumulative share, and assigns an A/B/C class — no manual sorting.
- Look at a slotting plan and calculate, in feet and in minutes, exactly what it costs (or saves) versus an alternative.
- Batch a set of orders into a wave and explain, with a number, why batch picking beats picking one order at a time.
- Size a picking crew from an order-volume target, an average pick time, and a shift length — and state the three numbers ("throughput," "cycle time," "productivity") a fulfillment manager reports every single day.

## Up next

Week 9 — Network Design & Facility Location: once you can run a single DC efficiently, the next question is *how many* DCs a network needs, and where.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
