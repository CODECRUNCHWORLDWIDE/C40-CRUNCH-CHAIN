# Week 7 — Logistics & Transportation Analytics

> **Goal:** by Sunday you can take a set of shipments and a set of delivery stops and answer the four questions a logistics analyst is paid to answer — *what does this lane cost, which mode should carry it, what's the cheapest route to visit these stops, and which carriers are actually earning their freight spend* — computing every one of them in SQL and Python, never a spreadsheet.

Welcome to **C40 · Crunch Chain**, Week 7. Weeks 5–6 sized *how much* inventory to hold and *who* to buy it from. This week is about the piece in between: getting product to physically move, at the lowest defensible cost, without breaking the service promise a customer was given. **Crunch Gear** ships from three regional distribution centers — **Austin East**, **Memphis DC**, and **Reno DC** — to wholesale accounts, its own DTC customers, and each other for replenishment, plus one long-haul import lane from its contract manufacturer's port in Hai Phong, Vietnam. Six modes, a dozen carriers, and a freight bill that adds up fast if nobody's watching it. That's this week's job.

We work against two seed datasets all week: a **124-row `shipments` table** — six weeks of real freight movement across every mode Crunch Gear uses — and a **13-stop `delivery_stops` table** for the Austin East regional last-mile delivery run, the network you'll route in Lecture 2.

**Data rule for this course:** every table, every routing calculation, every "which carrier should we drop" question this week is done in **SQL (PostgreSQL 16, SQLite fallback)** and/or **Python (pandas)** — never a spreadsheet. Spreadsheets are covered separately in [C41 Crunch Excel](../../../C41-CRUNCH-EXCEL/); a freight ledger and a routing problem are both just data, and treating them as data — queryable, joinable, versioned — is what lets you re-run "what if we switched this lane to intermodal" in ten seconds instead of rebuilding a workbook.

## Learning objectives

By the end of this week, you will be able to:

- **Model** transportation cost by mode, lane, and weight/volume break — and compute cost-per-unit-shipped so mode and carrier choices are comparable on a level footing.
- **Select** carriers and modes on an explicit cost, speed, and reliability trade-off instead of "we've always used them."
- **Formulate** the vehicle routing problem (VRP) and its common variants, and solve a capacitated instance with the nearest-neighbor and Clarke-Wright savings heuristics.
- **Analyze** carrier on-time performance and freight spend directly from shipment-level data in SQL — no manual pivot tables.
- **Balance** transportation cost against service commitments across a multi-DC network, and defend a mode/carrier mix with numbers instead of instinct.

## Prerequisites

- Comfortable with `SELECT`, `WHERE`, `GROUP BY`, `HAVING`, joins, and window functions in SQL (Weeks 2–6 of this course, or [C33 Crunch SQL](../../../C33-CRUNCH-SQL/)).
- Comfortable reading and writing basic pandas: `read_sql`, `groupby`, `apply`, sorting, and simple loops — Lecture 2's routing heuristics are iterative and easiest to write in Python.
- Basic geometry: the distance formula (Pythagorean theorem) — Lecture 2 uses straight-line distance between (x, y) coordinates as a teaching simplification for the VRP.
- PostgreSQL 16+ **or** SQLite 3.35+, plus Python 3.10+ with `pandas` and `numpy`. See [`resources.md`](./resources.md) for install steps.

## Setup — seed the Week 7 shipment ledger and delivery network

Everything this week runs against two tables. Create both once, before Lecture 1.

**PostgreSQL:**

```bash
createdb crunch_chain_wk7
psql crunch_chain_wk7
```

**SQLite:**

```bash
sqlite3 crunch_chain_wk7.db
```

### Table 1 — `shipments`

Six weeks of freight movement across Crunch Gear's network: outbound wholesale/DTC lanes from Austin East, inter-DC replenishment between all three DCs, and the Hai Phong import lane. Run this (identical on both engines):

```sql
CREATE TABLE shipments (
    shipment_id           INTEGER PRIMARY KEY,
    ship_date             DATE    NOT NULL,
    origin                TEXT    NOT NULL,   -- DC or supplier location
    destination            TEXT    NOT NULL,   -- market or DC
    lane_id                TEXT    NOT NULL,   -- short lane code, e.g. 'AUS-DAL'
    carrier                TEXT    NOT NULL,
    mode                   TEXT    NOT NULL,   -- Parcel, LTL, FTL, Intermodal, Ocean, Air
    weight_lbs             NUMERIC NOT NULL,
    distance_miles         NUMERIC NOT NULL,
    freight_cost           NUMERIC NOT NULL,   -- $ actually billed, fuel surcharge included
    promised_transit_days  NUMERIC NOT NULL,
    actual_transit_days    NUMERIC NOT NULL,
    units_shipped          INTEGER NOT NULL
);
```

Now load the 124 rows — the full seed is in [`resources.md`](./resources.md#full-shipments-seed-data) to keep this page short, but here's a representative slice so you can see the shape:

```sql
INSERT INTO shipments VALUES
(1,'2025-01-07','Austin East','Dallas, TX','AUS-DAL','Yellowline Freight','LTL',4787,195,1135.29,2,2,905),
(2,'2025-01-06','Austin East','Houston, TX','AUS-HOU','Longhaul Truckload','FTL',24104,165,1761.03,1,1,4895),
(9,'2025-01-07','Memphis DC','Chicago, IL','MEM-CHI','Longhaul Truckload','FTL',41627,530,3132.66,1,0,8633),
(119,'2025-01-07','Hai Phong, Vietnam','Austin East','HPH-ATX','Pacific Rim Ocean Lines','Ocean',24909,8100,2143.74,34,35,5535),
(120,'2025-01-07','Hai Phong, Vietnam','Austin East','HPH-ATX','SkyBridge Air Cargo','Air',1383,8100,5237.30,4,4,307);
-- ... 119 more rows — full block in resources.md
```

Sanity check — this should print `124`:

```sql
SELECT COUNT(*) FROM shipments;
```

A shipment is **on time** when `actual_transit_days <= promised_transit_days`. Notice `mode` mixes on the *same* lane (e.g., `AUS-DAL` has both `Parcel` and `LTL` rows, `MEM-AUS` has `Parcel`, `FTL`, *and* `Intermodal`) — that's on purpose. Real lanes carry a blend of shipment sizes, and picking the right mode per shipment (not per lane) is exactly the skill Lecture 3 builds.

### Table 2 — `delivery_stops`

The Austin East regional last-mile network: twelve wholesale accounts within driving distance of the DC, plus the depot itself as stop `0`. Coordinates are **miles on a simplified Cartesian grid** centered on the depot (real routing software uses road-network distance from a mapping API; straight-line distance is close enough to teach the algorithm and is what we use this week).

```sql
CREATE TABLE delivery_stops (
    stop_id      INTEGER PRIMARY KEY,
    account_name TEXT    NOT NULL,
    x_miles      NUMERIC NOT NULL,
    y_miles      NUMERIC NOT NULL,
    demand_cases INTEGER NOT NULL   -- order size in cases; depot row is 0
);

INSERT INTO delivery_stops VALUES
(0, 'Austin East DC',                     0,   0,  0),
(1, 'Round Rock Trailhead Outfitters',    8,  15, 42),
(2, 'Cedar Park Basecamp Supply',        -5,  18, 35),
(3, 'Georgetown Trail Co-op',            12,  25, 28),
(4, 'San Marcos River Gear',              5, -20, 50),
(5, 'New Braunfels Alpine Traders',      10, -28, 33),
(6, 'San Antonio North Summit Sports',    2, -48, 60),
(7, 'San Antonio South Trailblazers',    -8, -52, 45),
(8, 'Bastrop Backcountry',               25,  -5, 22),
(9, 'Lockhart Outpost Gear',             18, -18, 18),
(10, 'Kyle Ridge Running Co',             3, -12, 30),
(11, 'Buda Trail Supply',                 0,  -8, 26),
(12, 'Pflugerville Peak Outdoors',       10,  10, 20);
```

Sanity check — should print `13`:

```sql
SELECT COUNT(*) FROM delivery_stops;
```

Total demand across the 12 accounts is **409 cases**. Each delivery truck in this week's exercises carries **120 cases**, so a single vehicle cannot serve the whole network — you need at least 4 routes. Keep that number in your pocket for Lecture 2.

## Weekly schedule

Adds up to roughly the course's **~28 hr/week full-time pace**.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|--------------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Modes, cost drivers, weight breaks | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | Cost per lane in SQL; cost-per-unit math | 0h | 1.5h | 0h | 0.5h | 1h | 0h | 3h |
| Wednesday | The VRP; nearest-neighbor and savings heuristics | 2h | 1.5h | 1h | 0.5h | 1h | 0h | 6h |
| Thursday | Carrier selection & freight analytics in SQL | 2h | 1h | 1h | 0.5h | 1h | 1h | 6.5h |
| Friday | Mode-selection trade-off; catch-up | 0h | 0h | 1h | 0.5h | 1h | 1.5h | 4h |
| Saturday | Mini-project — routing + mode plan | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **4h** | **5h** | **3h** | **3.5h** | **5h** | **5h** | **27.5h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-transportation-modes-and-cost.md](./lecture-notes/01-transportation-modes-and-cost.md) | Parcel, LTL, FTL, intermodal, ocean, air; cost drivers, weight breaks, cost-per-unit-shipped | 2h |
| 2 | [lecture-notes/02-routing-and-the-vehicle-routing-problem.md](./lecture-notes/02-routing-and-the-vehicle-routing-problem.md) | The VRP and its variants; nearest-neighbor and savings heuristics | 2h |
| 3 | [lecture-notes/03-carrier-selection-and-freight-analytics.md](./lecture-notes/03-carrier-selection-and-freight-analytics.md) | Scoring carriers on cost + on-time reliability; mode trade-offs; freight spend analysis in SQL | 2h |
| 4 | [exercises/exercise-01-cost-per-lane.md](./exercises/exercise-01-cost-per-lane.md) | Compute cost per lane, cost per lb, and cost per unit shipped | 1h |
| 5 | [exercises/exercise-02-nearest-neighbor-routing.md](./exercises/exercise-02-nearest-neighbor-routing.md) | Build a nearest-neighbor route (and a capacitated multi-truck version) in Python | 1.5h |
| 6 | [exercises/exercise-03-carrier-on-time-analysis.md](./exercises/exercise-03-carrier-on-time-analysis.md) | Score every carrier on on-time % and cost in SQL | 1h |
| 7 | [challenges/challenge-01-savings-algorithm-routing.md](./challenges/challenge-01-savings-algorithm-routing.md) | Implement the Clarke-Wright savings algorithm and beat nearest-neighbor | 1.5h |
| 8 | [challenges/challenge-02-mode-selection-tradeoff.md](./challenges/challenge-02-mode-selection-tradeoff.md) | Find the shipments that used the wrong mode and quantify the waste | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Solve a lowest-cost routing + mode plan; justify the carrier mix | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice tying lanes, routing, and carrier scoring together | 5h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Full seed data, official references, tools to install | — |

## By the end of this week you can…

- Explain why cost per pound *drops* as shipment weight crosses each mode's break point, and pick the mode that minimizes cost-per-unit for a given shipment size.
- State the classic VRP and name its common variants (capacitated, time windows, multi-depot, pickup-and-delivery) — and know which one a given business problem actually is.
- Build a nearest-neighbor route and a Clarke-Wright savings route by hand in Python, and explain in one sentence why savings usually beats nearest-neighbor.
- Write a single SQL query that ranks every carrier by on-time percentage and total freight spend, no spreadsheet pivot table involved.
- Look at a shipment record and say, with a number attached, whether the mode chosen for it was the right call.

## Up next

Week 8 — Warehousing: slotting, pick profiles, throughput, and labor — once freight lands at Austin East, we go inside the building and optimize how it moves through it.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
