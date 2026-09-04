# Week 5 — Inventory Management — EOQ & Safety Stock

> **Goal:** by Sunday you can take any SKU's cost and demand data and answer the four questions every inventory planner is paid to answer — *how much do we order, when do we order it, how much safety stock do we carry, and what does getting it wrong cost us* — computing all four in SQL and pandas, never a spreadsheet.

Welcome to **C40 · Crunch Chain**, Week 5. Weeks 3–4 taught you to forecast demand — a number with an error bar around it. This week starts turning that forecast into a stocking policy: a concrete, defensible rule for how much inventory to hold and when to reorder it. Every dollar sitting on a shelf is a dollar not doing anything else for the business, and every stockout is a lost sale or an angry customer — inventory management is the discipline of finding the point that minimizes both at once.

We keep using **Crunch Gear**, the fictional outdoor-apparel company from Week 1: fabric from mills, jackets sewn by contract manufacturers, product flowing through regional warehouses to wholesale and DTC customers. This week we zoom into one node — the **Austin East regional warehouse** — and its catalog of ten SKUs. You'll size an ordering and safety-stock policy for that catalog from first principles, then let the numbers tell you what a 95% service level actually costs.

**Data rule for this course:** every table, every computation, every "what if we raised the service level" question in this week is done in **SQL (PostgreSQL 16, SQLite fallback)** and/or **Python (pandas)** — never a spreadsheet. Spreadsheets are covered separately in [C41 Crunch Excel](../../../C41-CRUNCH-EXCEL/); this course models inventory as data because in a real operations job, that's exactly what it is: a table you query, not a grid you eyeball.

## Learning objectives

By the end of this week, you will be able to:

- **Break down** inventory cost into holding, ordering, and shortage cost — and compute the **Economic Order Quantity (EOQ)** that minimizes the sum of holding and ordering cost.
- **Set safety stock** from a target service level, given demand variability and lead-time variability, using the normal-distribution z-score method.
- **Compute reorder points** and choose between **(s,Q)**, **(R,S)**, and **base-stock** review policies depending on how a system tracks inventory and how often it reviews it.
- **Solve the single-period newsvendor problem** for perishable or one-shot inventory, where there is no "next cycle" to recover from a stockout or an overbuy.
- **Extend** stocking logic across a multi-echelon network and see, in dollars, the cost of positioning safety stock in the wrong place.

## Standards this week meets

| Bar | What this week is measured against |
| --- | --- |
| University | `ISM 4400` — determine order quantity from the holding-versus-ordering trade-off, set safety stock and reorder points to a target service level, and solve the single-period newsvendor decision. |
| Industry | Set the stocking policy for a whole catalogue at a stated service level and answer the budget question that follows it: what does running the catalogue at that level cost the business, and what would one more point of service add on top. |
| Beyond the bar | The designed policy is simulated against the demand history to check empirically that it delivers the service level it was sized for, instead of trusting the formula that produced it — `exercises/exercise-03-simulate-a-reorder-point.md` |

## Prerequisites

- Comfortable with `SELECT`, `JOIN`, `GROUP BY`, and basic aggregate functions in SQL (Weeks 2–3 of this course, or [C33 Crunch SQL](../../../C33-CRUNCH-SQL/)).
- Comfortable reading and writing basic pandas: `read_sql`, column arithmetic, `groupby`, `apply`.
- A calculator (mental or otherwise) and comfort with square roots and basic algebra — the EOQ and safety-stock formulas are simple once you've derived them once.
- PostgreSQL 16+ **or** SQLite 3.35+, plus Python 3.10+ with `pandas`, `numpy`, and `scipy`. See [`resources.md`](./resources.md) for install steps.

## Setup — seed the Week 5 catalog

Every lecture, exercise, and challenge this week (except Challenge 2, which adds its own small dataset) works against one table: the ten-SKU catalog for the Austin East warehouse.

**PostgreSQL:**

```bash
createdb crunch_chain_wk5
psql crunch_chain_wk5
```

**SQLite:**

```bash
sqlite3 crunch_chain_wk5.db
```

Then run this (identical on both engines):

```sql
CREATE TABLE skus (
    sku_id            INTEGER PRIMARY KEY,
    sku_name          TEXT    NOT NULL,
    category          TEXT    NOT NULL,
    unit_cost         NUMERIC NOT NULL,   -- $ per unit, what Crunch Gear pays its supplier
    annual_demand     NUMERIC NOT NULL,   -- D, units/year, from the Week 3-4 forecast
    order_cost        NUMERIC NOT NULL,   -- S, $ per purchase order placed
    holding_pct       NUMERIC NOT NULL,   -- i, annual holding cost as a fraction of unit_cost
    lead_time_days    NUMERIC NOT NULL,   -- L, mean supplier lead time in days
    demand_std_daily  NUMERIC NOT NULL,   -- sigma_d, std dev of DAILY demand, units
    lead_time_std_days NUMERIC NOT NULL   -- sigma_L, std dev of lead time, days
);

INSERT INTO skus VALUES
(1,'Alpine Shell Jacket',        'Jackets',     84.00, 4800, 120, 0.25, 12,  9.5, 2.0),
(2,'Summit Down Parka',          'Jackets',    165.00, 1500, 120, 0.25, 18,  4.2, 3.0),
(3,'Trailhead Softshell Vest',   'Jackets',     58.00, 3200, 120, 0.25, 12,  6.8, 2.0),
(4,'Merino Base Layer Top',      'Base Layers', 42.00, 9600,  90, 0.22, 10, 14.5, 1.5),
(5,'Merino Base Layer Bottom',   'Base Layers', 38.00, 7200,  90, 0.22, 10, 11.2, 1.5),
(6,'Thermal Fleece Hoodie',      'Base Layers', 55.00, 5400,  90, 0.22, 14,  9.0, 2.0),
(7,'Crunch Trail Backpack 32L',  'Accessories', 72.00, 2600, 100, 0.20, 21,  5.4, 4.0),
(8,'Insulated Water Bottle',     'Accessories', 18.00,12000,  60, 0.20, 15, 22.0, 2.0),
(9,'Trekking Pole Pair',         'Accessories', 46.00, 3600, 100, 0.20, 18,  6.9, 3.0),
(10,'Wool Beanie',               'Accessories', 14.00, 8400,  60, 0.18,  9, 16.0, 1.0);
```

Sanity check — should print `10`:

```sql
SELECT COUNT(*) FROM skus;
```

`holding_pct` varies by category — Finance charges Jackets a higher carrying rate (0.25) than Accessories (0.18–0.20) because jackets tie up more capital per unit and carry more obsolescence risk (fashion/season risk) than a wool beanie or a water bottle. This is a normal, realistic thing for a holding-cost rate to do — don't assume it's a single constant across a whole catalog.

## Weekly schedule

Adds up to roughly the course's **~28 hr/week full-time pace**.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|--------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Inventory costs; deriving and computing EOQ | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | Safety stock, service levels, z-scores | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Wednesday | Reorder policies; (s,Q) vs (R,S) vs base-stock | 2h | 1h | 1h | 0.5h | 1h | 0h | 5.5h |
| Thursday | Newsvendor model; simulate a reorder policy | 0h | 1.5h | 1h | 0.5h | 1h | 1h | 5h |
| Friday | Multi-echelon positioning; challenges | 0h | 0h | 1h | 0.5h | 1h | 1.5h | 4h |
| Saturday | Mini-project — catalog-wide stocking policy | 0h | 0h | 0h | 0h | 0h | 3h | 3h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **5h** | **3h** | **3.5h** | **5h** | **5.5h** | **28h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-inventory-costs-and-eoq.md](./lecture-notes/01-inventory-costs-and-eoq.md) | Holding, ordering, shortage cost; deriving and computing EOQ; total-cost sensitivity | 2h |
| 2 | [lecture-notes/02-safety-stock-and-service-levels.md](./lecture-notes/02-safety-stock-and-service-levels.md) | Cycle-service level vs. fill rate, demand-during-lead-time variability, sizing safety stock to a z-target | 2h |
| 3 | [lecture-notes/03-reorder-policies-and-the-newsvendor.md](./lecture-notes/03-reorder-policies-and-the-newsvendor.md) | (s,Q), (R,S), base-stock policies; the newsvendor model for single-period decisions | 2h |
| 4 | [exercises/exercise-01-compute-eoq.md](./exercises/exercise-01-compute-eoq.md) | Compute EOQ, order frequency, and total cost for the full catalog in SQL + pandas | 1h |
| 5 | [exercises/exercise-02-safety-stock-from-service-level.md](./exercises/exercise-02-safety-stock-from-service-level.md) | Compute safety stock and reorder point at 90/95/99% service levels | 1.5h |
| 6 | [exercises/exercise-03-simulate-a-reorder-point.md](./exercises/exercise-03-simulate-a-reorder-point.md) | Simulate a year of daily demand under an (s,Q) policy; measure the realized service level | 1.5h |
| 7 | [challenges/challenge-01-newsvendor-optimal-order.md](./challenges/challenge-01-newsvendor-optimal-order.md) | Size a one-shot order for a limited-run product under demand uncertainty | 1h |
| 8 | [challenges/challenge-02-multi-echelon-inventory.md](./challenges/challenge-02-multi-echelon-inventory.md) | Centralize vs. decentralize safety stock across 3 warehouses; quantify the risk-pooling benefit | 2h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Set EOQ, ROP, and safety-stock policy for the full catalog to hit 95% service level at minimum cost | 3h |
| 10 | [homework.md](./homework.md) | Extra practice tying EOQ, safety stock, and reorder policy together | 5h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official/free references + tools to install | — |

## By the end of this week you can…

- Compute EOQ for any SKU from its demand, ordering cost, and holding cost, and explain why the total-cost curve is famously flat near the optimum.
- Turn a target service level into a z-score, a z-score into safety stock, and safety stock into a reorder point.
- Pick between (s,Q), (R,S), and base-stock review policies for a given inventory-tracking reality, and explain why periodic review needs more safety stock than continuous review.
- Solve a newsvendor problem for a perishable or one-shot product from a critical ratio.
- Explain, with numbers, why pooling safety stock centrally can serve the same demand with less total inventory than holding it at every location separately.

## Up next

[Week 6 — Procurement, spend analysis, supplier scorecards & lead-time risk](../week-06-procurement-spend-analysis-and-supplier-risk/) — now that you can size *how much* to hold, we turn to *who* you buy it from and what their reliability costs you.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
