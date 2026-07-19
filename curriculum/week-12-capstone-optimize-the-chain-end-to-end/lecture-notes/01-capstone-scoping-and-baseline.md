# Capstone Scoping & Baseline

Every real optimization project starts the same way, and it is not with a solver. It starts with a **scoping document** — a one-page statement of what network you're looking at, what's in scope to change, what's held fixed, what "better" means, and what constraint you're not allowed to break while chasing it. Skip this step and you'll spend two weeks writing beautiful linear programs that optimize the wrong thing, or that a VP rejects in the first sentence because you quietly assumed away the one constraint they actually care about. This lecture writes that scoping document for the capstone, then builds the SQL network it describes, then hand-computes a baseline so precise you could defend every digit of it in a room full of skeptical finance people. Lecture 2 automates everything this lecture does by hand.

## 1. The network, in one paragraph

**Crunch Gear** is expanding its Alpine jacket line. Product is made at two plants — an overseas contract mill in **Hai Phong, Vietnam** (`HPH`, low unit cost, 33–37 day ocean lead time) and a domestic cut-and-sew shop in **Statesville, NC** (`PCS`, higher unit cost, 4–9 day truck lead time) — and flows into three distribution centers you've seen since Week 1: **Austin East** (`AUS`), **Memphis DC** (`MEM`), and **Reno DC** (`REN`). From there it ships to four demand regions — **Northeast**, **Southeast**, **Midwest**, **West** — each anchored by a wholesale customer (TrailStop Outfitters, Ridgeline Retail, Prairie Supply Co., and Summit & Sea, respectively). Six SKUs move through this network: a rain shell, a down parka, a softshell, a fleece, a windbreaker, and a 3-in-1.

That's two plants, three DCs, four regions, six SKUs — a genuinely multi-echelon network, and small enough that you can hold the whole thing in your head while you check the solver's work.

## 2. The scoping document

Write this down before touching SQL. Here is the version for this capstone — copy the shape, not the specifics, into your own mini-project scoping doc.

**Problem statement.** Crunch Gear's DC-to-region assignment and safety-stock policy grew organically as DCs opened over the years, not from any cost analysis. We suspect it's leaving money on the table.

**In scope.** (1) Which DC serves which region, and in what split, subject to each DC's monthly throughput capacity. (2) Each SKU-region's safety-stock policy — currently a flat "30 days of extra supply," regardless of that SKU-region's actual demand volatility or lead time.

**Out of scope, held fixed.** Plant-to-DC sourcing mix (which plant fills which DC) does not change in this capstone — that's a separate sourcing-strategy question with contract and tariff implications beyond a flow-optimization exercise. Production unit costs and inbound freight costs are therefore held constant between baseline and optimized scenarios; only outbound freight, DC fixed operating cost, and safety-stock holding cost are in play. **Say this explicitly in your own write-ups** — a reviewer's first question is always "what did you hold constant, and why."

**Objective.** Minimize total monthly network cost within scope (outbound freight + DC fixed cost + safety-stock holding cost).

**Constraint.** Every region's monthly demand must be met in full by DC capacity (no region goes unserved), and the cycle service level implied by each SKU-region's safety-stock policy must be **at or above 95%**.

**Target.** Cut in-scope monthly network cost by **at least 6%** versus the current baseline, without breaching the 95% service constraint.

**Deliverable.** One reproducible pipeline (SQL + Python) that recomputes the baseline, runs the optimization, and prints the recommendation — re-runnable the moment new demand data lands, not a one-time spreadsheet exercise.

Notice what this scoping document did: it turned a vague ask ("optimize the supply chain") into a specific, falsifiable, boundaried question. That's the actual skill. A solver can answer almost anything you hand it — the hard part is deciding what to hand it.

## 3. Build the network schema in SQL

Everything below runs unchanged on PostgreSQL and SQLite unless noted.

```sql
CREATE TABLE plants (
    plant_id    TEXT PRIMARY KEY,
    plant_name  TEXT NOT NULL,
    location    TEXT NOT NULL
);
INSERT INTO plants VALUES
('HPH','Hai Phong Mill','Hai Phong, Vietnam'),
('PCS','Piedmont Cut & Sew','Statesville, NC, USA');

CREATE TABLE dcs (
    dc_id            TEXT PRIMARY KEY,
    dc_name          TEXT NOT NULL,
    location         TEXT NOT NULL,
    monthly_capacity INTEGER NOT NULL,   -- units/month, all SKUs combined
    fixed_cost       NUMERIC NOT NULL    -- $/month, lease + labor + utilities
);
INSERT INTO dcs VALUES
('AUS','Austin East','Austin, TX',   26000, 42000),
('MEM','Memphis DC',  'Memphis, TN', 22000, 38000),
('REN','Reno DC',     'Reno, NV',    15000, 31000);

CREATE TABLE regions (
    region_id     TEXT PRIMARY KEY,
    region_name   TEXT NOT NULL,
    anchor_customer TEXT NOT NULL
);
INSERT INTO regions VALUES
('NE','Northeast','TrailStop Outfitters'),
('SE','Southeast','Ridgeline Retail'),
('MW','Midwest',  'Prairie Supply Co.'),
('WE','West',     'Summit & Sea');

CREATE TABLE skus (
    sku_id      TEXT PRIMARY KEY,
    sku_name    TEXT NOT NULL,
    demand_share NUMERIC NOT NULL   -- this SKU's share of every region's total demand
);
INSERT INTO skus VALUES
('SKU-100','Trailhead Rain Shell', 0.20),
('SKU-200','Summit Down Parka',    0.15),
('SKU-300','Ridgeline Softshell',  0.18),
('SKU-400','Basecamp Fleece',      0.17),
('SKU-500','Alpine Windbreaker',   0.15),
('SKU-600','Cascade 3-in-1',       0.15);
-- sanity check: SELECT SUM(demand_share) FROM skus;  -->  1.00

CREATE TABLE unit_costs (
    plant_id    TEXT REFERENCES plants(plant_id),
    sku_id      TEXT REFERENCES skus(sku_id),
    unit_cost   NUMERIC NOT NULL,   -- $/unit, production only
    PRIMARY KEY (plant_id, sku_id)
);
INSERT INTO unit_costs VALUES
('HPH','SKU-100',18),('HPH','SKU-200',34),('HPH','SKU-300',22),
('HPH','SKU-400',16),('HPH','SKU-500',14),('HPH','SKU-600',29),
('PCS','SKU-100',25),('PCS','SKU-200',47),('PCS','SKU-300',30),
('PCS','SKU-400',22),('PCS','SKU-500',19),('PCS','SKU-600',40);

CREATE TABLE plant_dc_lanes (
    plant_id     TEXT REFERENCES plants(plant_id),
    dc_id        TEXT REFERENCES dcs(dc_id),
    freight_cost NUMERIC NOT NULL,   -- $/unit
    lead_time_days INTEGER NOT NULL,
    PRIMARY KEY (plant_id, dc_id)
);
INSERT INTO plant_dc_lanes VALUES
('HPH','AUS',3.10,35),('HPH','MEM',3.40,37),('HPH','REN',2.60,33),
('PCS','AUS',1.80,6), ('PCS','MEM',1.10,4), ('PCS','REN',2.90,9);

CREATE TABLE dc_region_lanes (
    dc_id        TEXT REFERENCES dcs(dc_id),
    region_id    TEXT REFERENCES regions(region_id),
    freight_cost NUMERIC NOT NULL,   -- $/unit
    lead_time_days INTEGER NOT NULL,
    PRIMARY KEY (dc_id, region_id)
);
INSERT INTO dc_region_lanes VALUES
('AUS','NE',2.40,3),('AUS','SE',1.30,2),('AUS','MW',1.70,2),('AUS','WE',2.10,3),
('MEM','NE',1.60,2),('MEM','SE',1.20,1),('MEM','MW',1.40,1),('MEM','WE',2.60,3),
('REN','NE',2.90,4),('REN','SE',2.70,4),('REN','MW',2.20,3),('REN','WE',1.10,1);
```

Sanity checks — run all four:

```sql
SELECT COUNT(*) FROM dcs;              --  3
SELECT COUNT(*) FROM dc_region_lanes;  -- 12  (3 DCs × 4 regions, every lane priced)
SELECT COUNT(*) FROM plant_dc_lanes;   --  6  (2 plants × 3 DCs)
SELECT SUM(demand_share) FROM skus;    -- 1.00
```

## 4. Generate 24 months of demand — in Python, reproducibly

A real capstone rarely gets 24 clean months of history handed to it as a single CSV. It's common to receive a shorter history and have to extend or simulate the rest for planning purposes — and doing that **transparently, with a documented, seeded generator**, is a legitimate and teachable technique, not a shortcut. The alternative — quietly typing 576 rows of demand into a spreadsheet by hand — is exactly the kind of untraceable, unauditable data entry this course's data rule exists to prevent.

Each region has an average monthly demand (summed across all six SKUs): Northeast 7,000, Southeast 9,000, Midwest 8,000, West 6,500. Jackets are seasonal — demand peaks August through December (back-to-school through holiday) and troughs in spring.

```python
import numpy as np
import pandas as pd
from sqlalchemy import create_engine

np.random.seed(40)  # reproducible — everyone who runs this gets the same numbers

REGION_BASE = {"NE": 7000, "SE": 9000, "MW": 8000, "WE": 6500}
SKU_SHARE = {"SKU-100": 0.20, "SKU-200": 0.15, "SKU-300": 0.18,
             "SKU-400": 0.17, "SKU-500": 0.15, "SKU-600": 0.15}
MONTHS = pd.period_range("2024-01", periods=24, freq="M")

# seasonal index by calendar month: Aug-Nov peak, Mar trough — averages to ~1.00
# across the year, so REGION_BASE above already *is* each region's annual mean
SEASONAL_INDEX = {1: 0.78, 2: 0.73, 3: 0.68, 4: 0.71, 5: 0.82, 6: 0.91,
                   7: 1.00, 8: 1.19, 9: 1.23, 10: 1.28, 11: 1.42, 12: 1.23}

rows = []
for region, base in REGION_BASE.items():
    for sku, share in SKU_SHARE.items():
        sku_region_base = base * share
        for month in MONTHS:
            seasonal = SEASONAL_INDEX[month.month]
            noise = np.random.normal(loc=1.0, scale=0.08)   # ±8% demand noise
            units = max(0, round(sku_region_base * seasonal * noise))
            rows.append({"region_id": region, "sku_id": sku,
                         "month": str(month), "units": units})

demand_history = pd.DataFrame(rows)
assert len(demand_history) == 4 * 6 * 24   # 576 rows

engine = create_engine("postgresql:///crunch_chain_capstone")   # or sqlite:///crunch_chain_capstone.db
demand_history.to_sql("demand_history", engine, if_exists="replace", index=False)
```

```sql
SELECT COUNT(*) FROM demand_history;   -- 576  (4 regions × 6 SKUs × 24 months)
```

**Why generate rather than hand-type this table:** 576 hand-typed rows are exactly the kind of data entry where a transposed digit hides for weeks. A seeded generator is inspectable (read the formula, know exactly what's in the table), reproducible (anyone re-running the script gets the same data), and — critically — it's the same posture you'll take on a real team when a stakeholder asks "can you also model what happens if demand ran 20% hotter?" You change one line and rerun, instead of re-typing a spreadsheet.

## 5. Profile the baseline — by hand, once

Before writing a single line of optimization code, compute the current-state cost with a calculator. You need this number to know whether the LP in Lecture 2 actually helped, and you need to be able to defend it without the solver in the room.

**Today's DC-to-region assignment** (organic, not cost-driven): Austin East — Crunch Gear's original 2016 DC — serves whatever it has room for; Reno DC picks up the overflow that's geographically closest to it; Memphis DC, opened later, has never been folded into the routing rule and sits mostly idle.

- **AUS** serves NE + SE + MW = 7,000 + 9,000 + 8,000 = **24,000 units/mo** (under its 26,000 cap)
- **REN** serves WE = **6,500 units/mo** (well under its 15,000 cap)
- **MEM** serves **0 units/mo** — but its $38,000/month fixed cost is still being paid

Outbound freight cost at the average month's volume:

| Lane | Volume | $/unit | Cost |
|------|-------:|-------:|-----:|
| AUS → NE | 7,000 | 2.40 | $16,800 |
| AUS → SE | 9,000 | 1.30 | $11,700 |
| AUS → MW | 8,000 | 1.70 | $13,600 |
| REN → WE | 6,500 | 1.10 | $7,150 |
| **Total outbound** | 30,500 | | **$49,250** |

Add fixed DC cost (all three DCs are staffed and leased regardless of volume): $42,000 + $38,000 + $31,000 = **$111,000/mo**.

**Naive safety stock:** Crunch Gear's current policy is a flat 30 days of extra supply held at whichever DC serves a region — no adjustment for that SKU-region's actual demand volatility or lead time. Thirty days of buffer, at the current average monthly volume of 30,500 units, is **30,500 units of safety stock**, network-wide (a full extra month sitting in DCs at all times). At a blended unit cost of $25 and a 22%/year holding rate:

```
monthly holding cost = units × unit_cost × (annual_rate / 12)
                      = 30,500 × 25 × (0.22 / 12)
                      = 30,500 × 25 × 0.018333
                      = $13,979/mo
```

### Baseline total (in scope)

| Component | Monthly cost |
|---|---:|
| Outbound freight | $49,250 |
| DC fixed cost | $111,000 |
| Safety-stock holding cost | $13,979 |
| **Baseline total (in scope)** | **$174,229/mo** |

Everything above is arithmetic you can redo on a phone calculator in five minutes. That's the point — a capstone recommendation that only exists inside a solver's output is a recommendation nobody can trust. Lecture 2 builds the pipeline that replaces this hand work at scale, and re-derives this exact $174,229 baseline as its starting point before optimizing against it.

## 6. What "done" looks like this week

By Wednesday you'll have a network LP that reassigns DC-to-region flow and a right-sized safety-stock policy running against this same schema, and by Friday a one-page memo stating exactly how much of the $174,229/mo baseline you cut, and by which two levers. Keep this lecture's numbers — $174,229/mo baseline, $49,250 outbound, $111,000 fixed, $13,979 holding — pinned somewhere visible. Every later file in this week checks its work against them.
