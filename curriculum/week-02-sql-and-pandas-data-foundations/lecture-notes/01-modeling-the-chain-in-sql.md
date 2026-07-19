# Modeling the Chain in SQL

Last week you drew Crunch Gear's network on paper: suppliers feeding two contract-manufacturing plants, plants shipping finished jackets to four regional distribution centers, DCs fulfilling wholesale orders from four customers. That drawing is correct, and it is also useless to a computer. This lecture turns it into a schema — a set of `CREATE TABLE` statements with primary keys, foreign keys, and constraints — so that the network can be queried, not just described.

By the end of this lecture you'll have designed (and understand every line of) the schema Exercise 1 has you actually build and seed. Read this lecture with that exercise open in a second tab; every table here is one you'll type in an hour.

## 1. Why the database, not a spreadsheet, is the system of record

Before any table design: a definition. The **system of record** is wherever the one true, current answer to "how many units of SKU X are in transit to the Reno DC right now" lives. Everything else — a report, a dashboard, an export — is a *view* of that answer, not the answer itself.

A spreadsheet fails at being a system of record for exactly the reasons it succeeds at being a quick scratch pad:

- **No enforced structure.** A cell can hold a number, a formula, a typo, or nothing, and the spreadsheet will not stop you. A database column typed `INTEGER NOT NULL` physically cannot hold `"aproximately 40"` or a blank.
- **No enforced relationships.** Nothing stops you from typing an order for `SKU-9999` that doesn't exist, or a shipment for an order that was deleted last week. A foreign key constraint makes that insert fail, loudly, the moment you try it — instead of silently producing a wrong report three months later.
- **No safe concurrent writes.** Two people editing the same sheet at once is a corruption risk. A database handles concurrent writes as its core job.
- **No real query language.** "Show me OTIF by region, only for orders that shipped from a DC that also handles SKU category 'outerwear'" is a five-line SQL query and a genuinely painful spreadsheet exercise involving several helper columns and a prayer.

None of this means spreadsheets are bad tools — they're excellent for a one-off calculation or a chart you'll throw away tomorrow (that's what C41 Crunch Excel teaches). It means they're the wrong tool for *storing the facts your business runs on*. That job belongs to a database with a schema that makes bad data structurally impossible to insert. That's what this lecture builds.

## 2. The entities: what actually needs its own table

Look back at Week 1's network map. Six kinds of "thing" showed up repeatedly, and each becomes one table:

| Entity | What it represents | Week 1 vocabulary |
|---|---|---|
| **Suppliers** | Upstream vendors Crunch Gear buys raw materials from | Tier-1/2 suppliers, the far-upstream nodes |
| **Sites** | Locations Crunch Gear itself operates — plants, DCs | The company's own echelon of nodes |
| **Lanes** | Transportation links between two nodes | The arrows in your network diagram |
| **SKUs** | Individual sellable products | "The jacket," "the fleece vest" |
| **Orders** | A wholesale customer's request for product | What starts the pull side of the chain |
| **Shipments** | An actual movement of product fulfilling an order | The physical trucks/containers/parcels moving |

Two of these split further once you look closely: an order can request several different SKUs (so you need an **order line** per SKU), and a shipment can carry several order lines at once (so you need a **shipment line** too). That's not over-engineering — it's how real freight works: a single truck routinely carries product for more than one order line, and a single order routinely ships in more than one truck when part of it is short or delayed. Modeling this with one flat table (like Week 1's simplified `order_lines` table, which assumed one line per order) works for a lecture example; it breaks the moment two orders share a shipment or one order splits across two.

## 3. Suppliers and sites

Suppliers and Crunch Gear's own facilities look similar (both are "a place with a name and a location") but they're different kinds of thing in an important way: Crunch Gear doesn't control a supplier's internal operations, only the lane that connects to it. Keep them as separate tables.

```sql
CREATE TABLE suppliers (
    supplier_id     SERIAL PRIMARY KEY,
    supplier_name   TEXT NOT NULL,
    category        TEXT NOT NULL,              -- 'fabric', 'trims', 'insulation', ...
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
```

Two design decisions worth pausing on:

- **`SERIAL PRIMARY KEY`.** `SERIAL` is Postgres shorthand for "an auto-incrementing integer" — insert a row without specifying an ID and Postgres assigns the next one. It's the primary key: unique, `NOT NULL` by definition, and what every other table will point back to. (SQLite's equivalent is `INTEGER PRIMARY KEY AUTOINCREMENT`, or often just `INTEGER PRIMARY KEY`, which auto-increments by default — see `resources.md` for the exact syntax difference between engines.)
- **One `sites` table, not three.** Plants, DCs, and stores share every column (name, region, city, country) and differ only in *role*. Modeling them as one table with a `site_type` column — instead of separate `plants`, `distribution_centers`, and `stores` tables — means a lane, an inventory record, or a query never has to know or care which kind of site it's pointing at; it just joins to `sites`. The `CHECK` constraint (`site_type IN (...)`) still enforces that only the three valid roles are allowed. This pattern — one table with a type discriminator column, instead of many near-identical tables — comes up constantly in real schemas; watch for it again in later weeks.

## 4. Lanes: the hardest table in this schema

A lane connects two nodes and carries cost, distance, transit time, and mode. The wrinkle: a lane's *origin* can be a supplier (inbound raw material) **or** a Crunch Gear site (an inter-facility move) — but never both, and never neither.

```sql
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
```

Walk through why this shape, and not something simpler, is necessary:

- **Why not one `origin_id` column?** Because a plain integer can't tell you *which table* it points into — `origin_id = 3` is ambiguous between supplier #3 and site #3. Two nullable foreign-key columns, one per possible origin type, keeps each reference unambiguous and lets Postgres actually enforce it (`REFERENCES suppliers(supplier_id)` and `REFERENCES sites(site_id)` are real, checkable constraints; a single untyped `origin_id` column can't reference two tables at once).
- **Why the `one_origin_only` `CHECK`?** Two nullable columns alone would silently allow a lane with both origins filled in, or neither — nonsense states no supply chain actually has. The `CHECK` constraint is an **XOR pattern**: exactly one of the two columns must be non-`NULL`. Try to insert a lane with both origins set, or both `NULL`, and Postgres rejects the row at insert time. This is the constraint doing real work: it makes an entire category of bad data — a lane that comes from nowhere, or from two places at once — physically impossible to store, instead of a bug you find three weeks later in a report.
- **Why is `dest_site_id` always a plain, single, `NOT NULL` column?** Because a lane's *destination* in this network is always a Crunch Gear site — you never ship product *to* a supplier. Only the origin needs the either/or shape.

## 5. SKUs, orders, and order lines

```sql
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
```

Notice the pattern: `orders` holds facts about the *order itself* (who, when, promised, from which DC) exactly once. `order_lines` holds one row **per SKU on that order**, each pointing back to its parent order with `order_id`. This is the standard **header/line** pattern — you'll see it again for `shipments`/`shipment_lines` below, and it's the same shape a real ERP's sales-order tables use. A `UNIQUE (order_id, sku_id)` constraint stops a data-entry mistake from creating two separate lines for the same SKU on the same order — if a customer wants more of a SKU, that's a bigger `qty_ordered` on the existing line, not a duplicate row.

`skus.plant_id NOT NULL REFERENCES sites(site_id)` says every SKU is produced at exactly one plant in this schema — a deliberate simplification (real companies dual-source SKUs across plants) that keeps this week's queries readable. `unit_price >= unit_cost` is a sanity constraint: it doesn't guarantee correct data, but it does reject an obviously wrong one (selling below cost would need an explicit, deliberate override elsewhere, not a typo slipping through).

## 6. Shipments and shipment lines

```sql
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
    shipment_id         INTEGER NOT NULL REFERENCES shipments(shipment_id),
    order_line_id       INTEGER NOT NULL REFERENCES order_lines(order_line_id),
    qty_shipped          INTEGER NOT NULL CHECK (qty_shipped > 0),
    damaged_in_transit   BOOLEAN NOT NULL DEFAULT FALSE,
    invoice_accurate     BOOLEAN NOT NULL DEFAULT TRUE
);
```

This is where the header/line pattern earns its keep. An order can ship in more than one shipment (a partial ship now, the rest later), and a shipment can carry lines from... in this schema, one order at a time, but multiple SKUs within it — because `shipment_lines.order_line_id` points at an `order_lines` row, and one order can have several. If Crunch Gear later consolidates multiple *orders* onto one truck, `shipments.order_id` would need to move down onto `shipment_lines` too — a real schema evolution question worth sitting with, not answering blindly. For this week, one shipment always belongs to exactly one order.

`damaged_in_transit` and `invoice_accurate` are here deliberately, not as an afterthought: they're what let this week's data compute a **real** perfect order rate — on-time **and** in-full **and** damage-free **and** accurately invoiced, all four required, exactly like Week 1's mini-project — straight from operational data, instead of the simplified two-factor version most of Week 1's exercises used.

## 7. The inventory ledger: why a running balance needs its own table

None of the tables so far can answer "how many units of the Summit Shell Jacket were sitting in the Reno DC on March 15th?" You could try to derive it from `shipments` alone, but shipments only capture *outbound* movement — you'd have no idea what arrived from the plant in the first place. The fix is a dedicated **event ledger**: one row per inventory-moving event, ordered by date, that a window function can turn into a running balance (Lecture 2 does exactly this).

```sql
CREATE TABLE inventory_transactions (
    txn_id      SERIAL PRIMARY KEY,
    sku_id      INTEGER NOT NULL REFERENCES skus(sku_id),
    site_id     INTEGER NOT NULL REFERENCES sites(site_id),
    txn_date    DATE NOT NULL,
    txn_type    TEXT NOT NULL CHECK (txn_type IN ('receipt','shipment','adjustment')),
    qty_change  INTEGER NOT NULL
);
```

`qty_change` is signed: positive for a `receipt` (stock arriving from a plant), negative for a `shipment` (stock leaving to fulfill an order) or a downward `adjustment` (a cycle-count correction, damage write-off, and so on). This is a **ledger pattern**, not a "current balance" table — there is deliberately no `on_hand_qty` column anywhere in this schema. The balance at any point in time is always *computed* by summing the ledger up to that date, never stored and updated in place. That's a real design trade-off worth naming explicitly:

- **Storing a running balance directly** (an `inventory.on_hand_qty` column you update on every movement) is faster to read but loses history — you can answer "what's on hand right now" instantly but can't answer "what was on hand on March 15th" at all, and a single missed update silently desyncs the number from reality forever.
- **A ledger you sum on read** (what this schema does) is slightly more work per query but is always correct, always auditable, and can answer *any* historical point-in-time question — because nothing is ever overwritten, only appended to.

Real operations systems typically do both — a ledger as the source of truth, plus a cached current-balance table refreshed from it for speed — but understanding the ledger-first version is what makes the cached version make sense later, so that's where this course starts.

## 8. The full schema, and how the pieces connect

```sql
CREATE TABLE suppliers ( ... );              -- Section 3
CREATE TABLE sites ( ... );                  -- Section 3
CREATE TABLE lanes ( ... );                  -- Section 4  (origin: supplier XOR site; dest: site)
CREATE TABLE skus ( ... );                   -- Section 5  (plant_id -> sites)
CREATE TABLE orders ( ... );                 -- Section 5  (source_dc_id -> sites)
CREATE TABLE order_lines ( ... );            -- Section 5  (order_id -> orders, sku_id -> skus)
CREATE TABLE shipments ( ... );              -- Section 6  (order_id -> orders, lane_id -> lanes)
CREATE TABLE shipment_lines ( ... );         -- Section 6  (shipment_id -> shipments, order_line_id -> order_lines)
CREATE TABLE inventory_transactions ( ... ); -- Section 7  (sku_id -> skus, site_id -> sites)
```

Trace one path through it to see the whole chain in one sentence: a **supplier** ships fabric over a **lane** to a **plant** (a `site`), which produces a **SKU**; a customer places an **order**, which has one or more **order lines**, each for one SKU; a **shipment** moves over a **lane** from the order's source DC (a `site`), and its **shipment lines** record how much of each order line actually went out — separately from how much was originally ordered, which is exactly the gap that makes fill rate and OTIF meaningful numbers instead of trivially 100%.

Every foreign key here is also a promise: `order_lines.sku_id REFERENCES skus(sku_id)` means Postgres will refuse to insert an order line for a SKU that doesn't exist, full stop, no exceptions, no silent `#REF!` the way a broken spreadsheet formula fails quietly. That promise is the entire reason this schema — not a folder of CSVs, not a workbook — is what makes the database a *system of record* instead of just *a place data happens to sit*.

## 9. What this buys you starting today

Every query in the rest of this week, and every KPI in the rest of this course, is a `SELECT` against these nine tables. Lecture 2 joins across all of them to answer real business questions. Lecture 3 shows how to pull the results into pandas for further modeling — without ever losing the guarantee that the *database*, not the DataFrame, is the one true answer. Exercise 1 has you type this exact schema and load it with a full seed dataset — do that next, before Lecture 2, since every query from here on assumes that data exists.

**Next:** [Exercise 1 — Seed the Network Database](../exercises/exercise-01-seed-the-network-database.md), then [Lecture 2 — Operations Analytics in SQL](./02-operations-analytics-in-sql.md).
