# Exercise 1 — Seed the Network Database

**Goal:** Build the full schema from Lecture 1 and load it with a month of real-looking Crunch Gear network data — 3 suppliers, 6 sites, 12 lanes, 8 SKUs, 12 orders, 14 shipments, and an inventory ledger. Every later lecture, exercise, challenge, and the mini-project queries this exact dataset, so get it running correctly before moving on.

**Estimated time:** 1.5 hours (mostly execution and verification — the SQL is provided).

## Setup

Create a database and connect to it.

**PostgreSQL:**

```bash
createdb crunch_chain
psql crunch_chain
```

**SQLite:**

```bash
sqlite3 crunch_chain.db
```

## Part 1 — Create the schema

Paste this in full (works unchanged on both engines):

```sql
CREATE TABLE suppliers (
    supplier_id     SERIAL PRIMARY KEY,
    supplier_name   TEXT NOT NULL,
    category        TEXT NOT NULL,
    country         TEXT NOT NULL,
    lead_time_days  INTEGER NOT NULL CHECK (lead_time_days > 0)
);

CREATE TABLE sites (
    site_id     SERIAL PRIMARY KEY,
    site_name   TEXT NOT NULL,
    site_type   TEXT NOT NULL CHECK (site_type IN ('plant','dc','store')),
    region      TEXT NOT NULL,
    city        TEXT NOT NULL,
    country     TEXT NOT NULL
);

CREATE TABLE lanes (
    lane_id                SERIAL PRIMARY KEY,
    origin_supplier_id     INTEGER REFERENCES suppliers(supplier_id),
    origin_site_id         INTEGER REFERENCES sites(site_id),
    dest_site_id           INTEGER NOT NULL REFERENCES sites(site_id),
    mode                   TEXT NOT NULL CHECK (mode IN ('truck','rail','ocean','air','parcel')),
    distance_miles         NUMERIC NOT NULL CHECK (distance_miles > 0),
    standard_transit_days  INTEGER NOT NULL CHECK (standard_transit_days > 0),
    cost_per_unit          NUMERIC(10,2) NOT NULL CHECK (cost_per_unit >= 0),
    CONSTRAINT one_origin_only CHECK (
        (origin_supplier_id IS NOT NULL AND origin_site_id IS NULL) OR
        (origin_supplier_id IS NULL AND origin_site_id IS NOT NULL)
    )
);

CREATE TABLE skus (
    sku_id      SERIAL PRIMARY KEY,
    sku_code    TEXT NOT NULL UNIQUE,
    description TEXT NOT NULL,
    category    TEXT NOT NULL,
    unit_cost   NUMERIC(10,2) NOT NULL CHECK (unit_cost >= 0),
    unit_price  NUMERIC(10,2) NOT NULL CHECK (unit_price >= unit_cost),
    plant_id    INTEGER NOT NULL REFERENCES sites(site_id)
);

CREATE TABLE orders (
    order_id        SERIAL PRIMARY KEY,
    customer_name   TEXT NOT NULL,
    region          TEXT NOT NULL,
    order_date      DATE NOT NULL,
    promised_date   DATE NOT NULL CHECK (promised_date >= order_date),
    source_dc_id    INTEGER NOT NULL REFERENCES sites(site_id)
);

CREATE TABLE order_lines (
    order_line_id   SERIAL PRIMARY KEY,
    order_id        INTEGER NOT NULL REFERENCES orders(order_id),
    sku_id          INTEGER NOT NULL REFERENCES skus(sku_id),
    qty_ordered     INTEGER NOT NULL CHECK (qty_ordered > 0),
    UNIQUE (order_id, sku_id)
);

CREATE TABLE shipments (
    shipment_id     SERIAL PRIMARY KEY,
    order_id        INTEGER NOT NULL REFERENCES orders(order_id),
    lane_id         INTEGER NOT NULL REFERENCES lanes(lane_id),
    ship_date       DATE NOT NULL,
    delivery_date   DATE NOT NULL CHECK (delivery_date >= ship_date),
    carrier         TEXT NOT NULL
);

CREATE TABLE shipment_lines (
    shipment_line_id    SERIAL PRIMARY KEY,
    shipment_id          INTEGER NOT NULL REFERENCES shipments(shipment_id),
    order_line_id        INTEGER NOT NULL REFERENCES order_lines(order_line_id),
    qty_shipped           INTEGER NOT NULL CHECK (qty_shipped > 0),
    damaged_in_transit    BOOLEAN NOT NULL DEFAULT FALSE,
    invoice_accurate      BOOLEAN NOT NULL DEFAULT TRUE
);

CREATE TABLE inventory_transactions (
    txn_id      SERIAL PRIMARY KEY,
    sku_id      INTEGER NOT NULL REFERENCES skus(sku_id),
    site_id     INTEGER NOT NULL REFERENCES sites(site_id),
    txn_date    DATE NOT NULL,
    txn_type    TEXT NOT NULL CHECK (txn_type IN ('receipt','shipment','adjustment')),
    qty_change  INTEGER NOT NULL
);
```

**SQLite note:** SQLite accepts this as-is — it doesn't strictly enforce `CHECK`/type constraints by default unless foreign keys are turned on. Run `PRAGMA foreign_keys = ON;` right after connecting, before creating the tables, or the referential-integrity checks later in this exercise won't actually catch anything.

## Part 2 — Load the seed data

Paste each block in order (they reference each other, so order matters).

```sql
INSERT INTO suppliers (supplier_id, supplier_name, category, country, lead_time_days) VALUES
(1, 'Highland Textile Mills',  'fabric',     'Vietnam', 21),
(2, 'Northgate Zipper Co.',    'trims',      'China',   18),
(3, 'PurePoly Insulation',     'insulation', 'USA',     10);

INSERT INTO sites (site_id, site_name, site_type, region, city, country) VALUES
(1, 'Riverside Cut & Sew',        'plant', 'APAC',      'Ho Chi Minh City', 'Vietnam'),
(2, 'Border Assembly Plant',      'plant', 'LATAM',     'Monterrey',        'Mexico'),
(3, 'Reno DC',                    'dc',    'West',      'Reno',             'USA'),
(4, 'Columbus DC',                'dc',    'Midwest',   'Columbus',         'USA'),
(5, 'Newark DC',                  'dc',    'Northeast', 'Newark',           'USA'),
(6, 'Atlanta DC',                 'dc',    'Southeast', 'Atlanta',          'USA'),
(7, 'West Regional Depot',        'store', 'West',      'Reno',             'USA'),
(8, 'Midwest Regional Depot',     'store', 'Midwest',   'Columbus',         'USA'),
(9, 'Northeast Regional Depot',   'store', 'Northeast', 'Newark',           'USA'),
(10,'Southeast Regional Depot',   'store', 'Southeast', 'Atlanta',          'USA');

INSERT INTO lanes (lane_id, origin_supplier_id, origin_site_id, dest_site_id, mode, distance_miles, standard_transit_days, cost_per_unit) VALUES
(1,  1,    NULL, 1,  'ocean', 8500, 24, 0.15),
(2,  2,    NULL, 1,  'ocean', 1200, 12, 0.05),
(3,  3,    NULL, 2,  'truck', 1800,  4, 0.20),
(4,  NULL, 1,    3,  'ocean', 7200, 18, 3.10),
(5,  NULL, 1,    5,  'ocean', 8900, 21, 3.40),
(6,  NULL, 2,    4,  'truck', 1450,  3, 1.10),
(7,  NULL, 2,    6,  'truck', 1250,  3, 1.05),
(8,  NULL, 2,    3,  'rail',  1700,  5, 0.95),
(9,  NULL, 3,    7,  'truck',   25,  1, 0.40),
(10, NULL, 4,    8,  'truck',   20,  1, 0.35),
(11, NULL, 5,    9,  'truck',   18,  1, 0.35),
(12, NULL, 6,    10, 'truck',   22,  1, 0.35);

INSERT INTO skus (sku_id, sku_code, description, category, unit_cost, unit_price, plant_id) VALUES
(1, 'CG-JKT-001', 'Summit Shell Jacket',    'outerwear', 22.00, 42.00, 1),
(2, 'CG-JKT-002', 'Alpine Parka',           'outerwear', 26.00, 48.00, 1),
(3, 'CG-FLC-001', 'Ridge Fleece Pullover',  'midlayer',  14.00, 28.00, 2),
(4, 'CG-FLC-002', 'Basecamp Fleece Vest',   'midlayer',  11.00, 22.00, 2),
(5, 'CG-PNT-001', 'Trailhead Softshell Pant','bottoms',  18.00, 36.00, 1),
(6, 'CG-PNT-002', 'Canyon Cargo Pant',      'bottoms',   15.00, 30.00, 2),
(7, 'CG-ACC-001', 'Summit Beanie',          'accessory',  3.00,  8.00, 2),
(8, 'CG-ACC-002', 'Trail Gaiters',          'accessory',  5.00, 12.00, 1);

INSERT INTO orders (order_id, customer_name, region, order_date, promised_date, source_dc_id) VALUES
(1,  'TrailStop Outfitters', 'Northeast', '2026-03-02', '2026-03-09', 5),
(2,  'TrailStop Outfitters', 'Northeast', '2026-03-10', '2026-03-17', 5),
(3,  'Ridgeline Retail',     'Southeast', '2026-03-03', '2026-03-10', 6),
(4,  'Ridgeline Retail',     'Southeast', '2026-03-12', '2026-03-19', 6),
(5,  'Prairie Supply Co.',   'Midwest',   '2026-03-04', '2026-03-11', 4),
(6,  'Prairie Supply Co.',   'Midwest',   '2026-03-14', '2026-03-21', 4),
(7,  'Summit & Sea',         'West',      '2026-03-05', '2026-03-12', 3),
(8,  'Summit & Sea',         'West',      '2026-03-16', '2026-03-23', 3),
(9,  'TrailStop Outfitters', 'Northeast', '2026-03-20', '2026-03-27', 5),
(10, 'Ridgeline Retail',     'Southeast', '2026-03-22', '2026-03-29', 6),
(11, 'Prairie Supply Co.',   'Midwest',   '2026-03-24', '2026-03-31', 4),
(12, 'Summit & Sea',         'West',      '2026-03-26', '2026-04-02', 3);

INSERT INTO order_lines (order_line_id, order_id, sku_id, qty_ordered) VALUES
(1,  1,  1, 100),
(2,  1,  5,  50),
(3,  2,  2,  60),
(4,  3,  3,  80),
(5,  3,  7, 200),
(6,  4,  4,  70),
(7,  5,  3,  90),
(8,  5,  6,  40),
(9,  6,  8, 150),
(10, 7,  1, 120),
(11, 7,  5,  60),
(12, 8,  2,  45),
(13, 9,  1,  80),
(14, 10, 3, 100),
(15, 10, 4,  50),
(16, 11, 6,  65),
(17, 12, 2,  70),
(18, 12, 8,  90);

INSERT INTO shipments (shipment_id, order_id, lane_id, ship_date, delivery_date, carrier) VALUES
(1,  1,  11, '2026-03-07', '2026-03-09', 'SwiftHaul Logistics'),
(2,  2,  11, '2026-03-15', '2026-03-18', 'SwiftHaul Logistics'),
(3,  3,  12, '2026-03-08', '2026-03-09', 'Regional Freight Co.'),
(4,  3,  12, '2026-03-10', '2026-03-12', 'Regional Freight Co.'),
(5,  4,  12, '2026-03-17', '2026-03-18', 'Regional Freight Co.'),
(6,  5,  10, '2026-03-09', '2026-03-11', 'Heartland Carriers'),
(7,  6,  10, '2026-03-19', '2026-03-23', 'Heartland Carriers'),
(8,  7,  9,  '2026-03-10', '2026-03-12', 'Pacific Crest Trucking'),
(9,  8,  9,  '2026-03-21', '2026-03-22', 'Pacific Crest Trucking'),
(10, 9,  11, '2026-03-25', '2026-03-28', 'SwiftHaul Logistics'),
(11, 10, 12, '2026-03-27', '2026-03-28', 'Regional Freight Co.'),
(12, 10, 12, '2026-03-28', '2026-03-31', 'Regional Freight Co.'),
(13, 11, 10, '2026-03-29', '2026-03-30', 'Heartland Carriers'),
(14, 12, 9,  '2026-03-31', '2026-04-01', 'Pacific Crest Trucking');

INSERT INTO shipment_lines (shipment_line_id, shipment_id, order_line_id, qty_shipped, damaged_in_transit, invoice_accurate) VALUES
(1,  1,  1,  100, FALSE, TRUE),
(2,  1,  2,   50, FALSE, TRUE),
(3,  2,  3,   60, FALSE, TRUE),
(4,  3,  4,   80, FALSE, TRUE),
(5,  4,  5,  180, TRUE,  TRUE),
(6,  5,  6,   70, FALSE, TRUE),
(7,  6,  7,   90, FALSE, TRUE),
(8,  6,  8,   40, FALSE, TRUE),
(9,  7,  9,  150, FALSE, TRUE),
(10, 8,  10, 120, FALSE, TRUE),
(11, 8,  11,  60, FALSE, TRUE),
(12, 9,  12,  45, FALSE, TRUE),
(13, 10, 13,  80, FALSE, TRUE),
(14, 11, 14, 100, FALSE, TRUE),
(15, 12, 15,  45, FALSE, FALSE),
(16, 13, 16,  65, FALSE, TRUE),
(17, 14, 17,  70, FALSE, TRUE),
(18, 14, 18,  90, FALSE, TRUE);

INSERT INTO inventory_transactions (txn_id, sku_id, site_id, txn_date, txn_type, qty_change) VALUES
(1,  1, 5, '2026-03-01', 'receipt',    300),
(2,  1, 5, '2026-03-07', 'shipment',  -100),
(3,  1, 5, '2026-03-15', 'receipt',    150),
(4,  1, 5, '2026-03-25', 'shipment',   -80),
(5,  1, 5, '2026-03-29', 'adjustment',  -5),
(6,  1, 3, '2026-03-01', 'receipt',    250),
(7,  1, 3, '2026-03-10', 'shipment',  -120),
(8,  1, 3, '2026-03-20', 'receipt',    100),
(9,  1, 3, '2026-03-31', 'adjustment', -10),
(10, 3, 6, '2026-03-01', 'receipt',    200),
(11, 3, 6, '2026-03-08', 'shipment',   -80),
(12, 3, 6, '2026-03-18', 'receipt',    150),
(13, 3, 6, '2026-03-27', 'shipment',  -100),
(14, 3, 4, '2026-03-01', 'receipt',    180),
(15, 3, 4, '2026-03-09', 'shipment',   -90),
(16, 3, 4, '2026-03-22', 'receipt',    120);
```

## Part 3 — Verify the load

Run each check. Row counts must match exactly.

```sql
SELECT COUNT(*) FROM suppliers;               -- must print 3
SELECT COUNT(*) FROM sites;                   -- must print 10
SELECT COUNT(*) FROM lanes;                   -- must print 12
SELECT COUNT(*) FROM skus;                    -- must print 8
SELECT COUNT(*) FROM orders;                  -- must print 12
SELECT COUNT(*) FROM order_lines;             -- must print 18
SELECT COUNT(*) FROM shipments;               -- must print 14
SELECT COUNT(*) FROM shipment_lines;          -- must print 18
SELECT COUNT(*) FROM inventory_transactions;  -- must print 16
```

Now verify referential integrity holds — every one of these should return **zero rows**, because the foreign keys should have made orphans impossible to insert in the first place. Running them anyway is good habit: it's how you'd sanity-check a load from an external source that *didn't* go through your own `CREATE TABLE` statements.

```sql
-- Order lines pointing at a SKU that doesn't exist
SELECT * FROM order_lines ol
LEFT JOIN skus s ON ol.sku_id = s.sku_id
WHERE s.sku_id IS NULL;

-- Shipment lines pointing at an order line that doesn't exist
SELECT * FROM shipment_lines sl
LEFT JOIN order_lines ol ON sl.order_line_id = ol.order_line_id
WHERE ol.order_line_id IS NULL;

-- Lanes that somehow have both, or neither, origin type set
SELECT * FROM lanes
WHERE (origin_supplier_id IS NOT NULL AND origin_site_id IS NOT NULL)
   OR (origin_supplier_id IS NULL AND origin_site_id IS NULL);
```

## Part 4 — Prove the constraints actually work

Try each of these on purpose. Each one should **fail** with a constraint-violation error — that's the correct, desired outcome. Write down which constraint stopped each one.

1. Insert a `lanes` row with both `origin_supplier_id = 1` and `origin_site_id = 1` set.
2. Insert an `order_lines` row with `sku_id = 999` (doesn't exist).
3. Insert an `orders` row with `promised_date` earlier than `order_date`.
4. Insert a `sites` row with `site_type = 'warehouse'` (not one of the three allowed values).

## Done when…

- [ ] All nine `CREATE TABLE` statements ran without error.
- [ ] Every row-count check in Part 3 matches exactly.
- [ ] All three referential-integrity checks in Part 3 return zero rows.
- [ ] All four "should fail" inserts in Part 4 actually failed, and you can name which constraint stopped each one.

## Stretch

- Write one more `CHECK` constraint this schema is missing: `shipment_lines.qty_shipped` should probably never exceed the `order_lines.qty_ordered` it's fulfilling by an unreasonable amount — but a plain `CHECK` on `shipment_lines` alone can't see across to `order_lines`. Look up Postgres `CREATE TRIGGER` (or just write the SQL query that would *detect* an over-ship without a trigger) — you're not expected to implement it fully this week, just to see where a simple `CHECK` constraint stops being enough.

## Submission

Commit your full `schema.sql` (Parts 1–2 combined) to your portfolio under `c40-week-02/exercise-01/`. Keep this file — every remaining file this week assumes it's already loaded.
