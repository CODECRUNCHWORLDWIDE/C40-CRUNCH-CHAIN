# Week 6 — Procurement & Supplier Analytics

> **Goal:** by Sunday you can take a quarter's raw purchase-order data and turn it into a spend cube, a weighted supplier scorecard, and a total-cost-of-ownership comparison — then use all three to make and defend a real sourcing recommendation (consolidate, dual-source, or re-award) backed by numbers, not gut feel.

Welcome back to **C40 · Crunch Chain**. Procurement is where supply chain meets the outside world — every supplier relationship, every purchase order, every negotiated price is a decision made under incomplete information about a company you don't control. This week gives you the analytics to make those decisions with evidence: where the money actually goes (spend analysis), which suppliers actually perform (scorecards), what a part *really* costs once you look past the unit price (TCO), and how a supplier's unreliability quietly forces you to carry more inventory than the catalog price ever mentioned.

We continue with **Crunch Gear**, the outdoor-apparel company you've followed since Week 1. This week you sit inside its procurement team for a full fiscal quarter (Q1 2026, Jan–Mar), working from one running purchase-order dataset across fabric mills, contract manufacturers (CMT — cut, make, trim), trims suppliers, packaging vendors, and a scattering of small indirect purchases. That one dataset powers every lecture, exercise, challenge, and the mini-project.

**Data rule, unchanged:** every table, every cube, every score this week is built in **SQL (PostgreSQL 16, SQLite fallback) and/or Python (pandas)** — never a spreadsheet. Procurement is one of the business functions most commonly (and most dangerously) run out of Excel; this week shows you exactly why that habit breaks at scale and how to replace it.

## Learning objectives

By the end of this week, you will be able to:

- **Build a spend cube in SQL** — pivot purchase-order data by category, supplier, and business unit; find tail spend and maverick (off-contract) buying using Pareto analysis.
- **Score supplier performance** on cost, quality, on-time delivery, and lead-time reliability using a weighted scorecard built from real order-and-receipt data — and know the scorecard's blind spots (small sample sizes, single-metric gaming).
- **Model total cost of ownership (TCO)** beyond unit price — freight, quality/rework cost, and the carrying cost that a supplier's lead-time variability quietly forces onto your inventory.
- **Quantify lead-time risk** with a safety-stock formula, and trace its direct, dollar-denominated knock-on effect on inventory policy.
- **Recommend sourcing actions** — consolidation, dual-sourcing, or award reallocation — and defend them with a should-cost model and a documented set of assumptions.

## Prerequisites

- Comfortable with `SELECT`, `WHERE`, `GROUP BY`, `JOIN`, and basic aggregate functions (`SUM`, `AVG`, `COUNT`) in SQL — Week 2's foundations.
- Comfortable with pandas `groupby`, basic arithmetic on DataFrame columns, and reading a CSV/SQL table into a DataFrame.
- Week 1's KPI vocabulary (OTIF, fill rate) and Week 5's inventory/safety-stock concepts help — this week reuses both, applied to the *supply side* instead of the *demand side*.
- PostgreSQL 16+ or SQLite 3.35+ installed and reachable from a terminal. See [`resources.md`](./resources.md) if you need to (re)install.

## Setup

Everything this week runs against one purchase-order dataset. Create the database and load the seed once — every lecture and exercise below reuses it.

**PostgreSQL:**

```bash
createdb crunch_procurement
psql crunch_procurement
```

**SQLite:**

```bash
sqlite3 crunch_procurement.db
```

Then paste this into the shell (works unchanged on both engines):

```sql
CREATE TABLE purchase_orders (
    po_id           INTEGER PRIMARY KEY,
    supplier        TEXT    NOT NULL,
    category        TEXT    NOT NULL,   -- Fabric, CMT, Trims, Packaging, MRO / Indirect
    business_unit   TEXT    NOT NULL,   -- Outerwear, Accessories, Fulfillment & Ops
    order_date      DATE    NOT NULL,
    promised_date   DATE    NOT NULL,
    receipt_date    DATE    NOT NULL,
    qty_ordered     INTEGER NOT NULL,
    qty_received    INTEGER NOT NULL,
    unit_price      NUMERIC NOT NULL,
    defect_qty      INTEGER NOT NULL,   -- units received that failed inbound QC
    freight_cost    NUMERIC NOT NULL,
    maverick_buy    BOOLEAN NOT NULL    -- TRUE = off-contract / uncontrolled spend
);

INSERT INTO purchase_orders VALUES
(1, 'Alpine Weave Mills','Fabric','Outerwear','2026-01-05','2026-01-20','2026-01-19',5000,5000,18.20,25,1400,FALSE),
(2, 'Alpine Weave Mills','Fabric','Outerwear','2026-01-22','2026-02-06','2026-02-09',4200,4200,18.20,10,1180,FALSE),
(3, 'Alpine Weave Mills','Fabric','Outerwear','2026-02-10','2026-02-25','2026-02-24',4800,4750,18.20,15,1340,FALSE),
(4, 'Alpine Weave Mills','Fabric','Outerwear','2026-03-01','2026-03-16','2026-03-20',5200,5200,18.50,30,1460,FALSE),
(5, 'Everest Textile Co.','Fabric','Outerwear','2026-01-08','2026-01-29','2026-01-30',3000,2950,17.60,60,1050,FALSE),
(6, 'Everest Textile Co.','Fabric','Outerwear','2026-02-02','2026-02-23','2026-03-02',2800,2700,17.60,80,980,FALSE),
(7, 'Everest Textile Co.','Fabric','Outerwear','2026-02-27','2026-03-20','2026-03-19',3100,3050,17.80,45,1085,FALSE),
(8, 'Highland Fabric Group','Fabric','Outerwear','2026-01-15','2026-02-05','2026-02-04',1200,1200,19.90,5,520,FALSE),
(9, 'Highland Fabric Group','Fabric','Outerwear','2026-03-05','2026-03-26','2026-03-25',1400,1400,19.90,8,610,FALSE),
(10,'Andes Stitch Works','CMT','Outerwear','2026-01-10','2026-02-07','2026-02-10',1500,1480,22.40,12,2100,FALSE),
(11,'Andes Stitch Works','CMT','Outerwear','2026-02-01','2026-03-01','2026-03-04',1600,1560,22.40,20,2240,FALSE),
(12,'Andes Stitch Works','CMT','Outerwear','2026-02-20','2026-03-20','2026-03-19',1400,1400,22.60,9,1960,FALSE),
(13,'Pacific Rim Garments','CMT','Outerwear','2026-01-12','2026-02-09','2026-02-08',1200,1190,20.80,30,1680,FALSE),
(14,'Pacific Rim Garments','CMT','Outerwear','2026-02-05','2026-03-05','2026-03-11',1300,1250,20.80,42,1820,FALSE),
(15,'Pacific Rim Garments','CMT','Outerwear','2026-03-01','2026-03-29','2026-04-03',1250,1180,21.00,55,1750,FALSE),
(16,'Northstar Apparel Mfg','CMT','Accessories','2026-01-18','2026-02-15','2026-02-14',900,900,15.20,6,900,FALSE),
(17,'Northstar Apparel Mfg','CMT','Accessories','2026-02-15','2026-03-15','2026-03-14',950,940,15.20,8,950,FALSE),
(18,'ButtonWorks Supply','Trims','Outerwear','2026-01-06','2026-01-20','2026-01-18',20000,20000,0.42,120,340,FALSE),
(19,'ButtonWorks Supply','Trims','Outerwear','2026-02-10','2026-02-24','2026-02-27',18000,17900,0.42,150,310,FALSE),
(20,'ButtonWorks Supply','Trims','Accessories','2026-03-01','2026-03-15','2026-03-14',9000,9000,0.44,40,180,FALSE),
(21,'Zephyr Hardware Co.','Trims','Outerwear','2026-01-20','2026-02-03','2026-02-08',15000,14700,0.39,200,300,FALSE),
(22,'Zephyr Hardware Co.','Trims','Outerwear','2026-02-25','2026-03-11','2026-03-17',16000,15600,0.39,260,320,FALSE),
(23,'EcoPack Solutions','Packaging','Fulfillment & Ops','2026-01-09','2026-01-23','2026-01-22',12000,12000,0.85,10,600,FALSE),
(24,'EcoPack Solutions','Packaging','Fulfillment & Ops','2026-02-18','2026-03-04','2026-03-06',13000,12950,0.85,15,650,FALSE),
(25,'CartonWorks Inc','Packaging','Fulfillment & Ops','2026-01-14','2026-01-28','2026-01-27',10000,10000,0.90,5,520,FALSE),
(26,'CartonWorks Inc','Packaging','Fulfillment & Ops','2026-03-02','2026-03-16','2026-03-15',11000,11000,0.90,6,560,FALSE),
(27,'QuickFix Facilities','MRO / Indirect','Fulfillment & Ops','2026-01-11','2026-01-13','2026-01-14',1,1,780.00,0,0,TRUE),
(28,'Ace Industrial Parts','MRO / Indirect','Fulfillment & Ops','2026-01-25','2026-01-27','2026-01-30',1,1,415.00,0,0,TRUE),
(29,'OfficeSupply Direct','MRO / Indirect','Fulfillment & Ops','2026-02-03','2026-02-05','2026-02-05',1,1,290.00,0,0,TRUE),
(30,'Rapid Print Co','MRO / Indirect','Fulfillment & Ops','2026-02-14','2026-02-16','2026-02-18',1,1,610.00,0,0,TRUE),
(31,'QuickFix Facilities','MRO / Indirect','Fulfillment & Ops','2026-02-22','2026-02-24','2026-02-24',1,1,545.00,0,0,TRUE),
(32,'Bolt & Nut Supply Co','MRO / Indirect','Fulfillment & Ops','2026-03-03','2026-03-05','2026-03-09',1,1,380.00,0,0,TRUE),
(33,'Ace Industrial Parts','MRO / Indirect','Fulfillment & Ops','2026-03-12','2026-03-14','2026-03-13',1,1,720.00,0,0,TRUE),
(34,'OfficeSupply Direct','MRO / Indirect','Fulfillment & Ops','2026-03-20','2026-03-22','2026-03-25',1,1,260.00,0,0,TRUE);
```

Sanity check — this should print `34`:

```sql
SELECT COUNT(*) FROM purchase_orders;
```

And this should print `843610.00` — total Q1 spend across all 34 lines:

```sql
SELECT SUM(qty_ordered * unit_price) FROM purchase_orders;
```

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-spend-analysis-and-the-spend-cube.md](./lecture-notes/01-spend-analysis-and-the-spend-cube.md) | Building a spend cube in SQL, category taxonomies, tail spend, and maverick buying | 2h |
| 2 | [lecture-notes/02-supplier-performance-and-scoring.md](./lecture-notes/02-supplier-performance-and-scoring.md) | Weighted scorecards across cost, quality, on-time delivery, and lead-time variability | 2h |
| 3 | [lecture-notes/03-total-cost-of-ownership-and-sourcing.md](./lecture-notes/03-total-cost-of-ownership-and-sourcing.md) | TCO beyond unit price, single vs. multi-source trade-offs, lead-time risk feeding into inventory | 2h |
| 4 | [exercises/exercise-01-build-a-spend-cube-in-sql.md](./exercises/exercise-01-build-a-spend-cube-in-sql.md) | Pivot the seed data by category/supplier/BU/month; find tail spend | 1.5h |
| 5 | [exercises/exercise-02-build-a-supplier-scorecard.md](./exercises/exercise-02-build-a-supplier-scorecard.md) | Score the three Fabric suppliers on cost/quality/delivery/lead-time | 1.5h |
| 6 | [exercises/exercise-03-price-variance-analysis.md](./exercises/exercise-03-price-variance-analysis.md) | Compare actual paid price vs. contracted standard cost, by supplier | 1h |
| 7 | [challenges/challenge-01-supplier-award-allocation.md](./challenges/challenge-01-supplier-award-allocation.md) | Split next quarter's CMT volume across 3 capacity-constrained suppliers | 1.5h |
| 8 | [challenges/challenge-02-should-cost-model.md](./challenges/challenge-02-should-cost-model.md) | Build a bottom-up should-cost model and find the negotiation gap | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Spend cube + scorecard + lead-time risk + a defended sourcing recommendation | 3h |
| 10 | [homework.md](./homework.md) | Extra practice, spaced across the week | 4.5h |
| 11 | [quiz.md](./quiz.md) | 14 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official/free references + tools to install | — |

## Weekly schedule

Adds up to roughly the course's **~28 hr/week full-time pace**. Adjust to your own pace per the syllabus.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|--------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Spend cube, category taxonomy, tail spend | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Tuesday | Supplier scorecards | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Wednesday | TCO, price variance | 2h | 1h | 0h | 0.5h | 1h | 1h | 5.5h |
| Thursday | Lead-time risk → safety stock; award allocation | 0h | 0h | 1.5h | 0.5h | 1h | 1h | 4h |
| Friday | Should-cost modeling | 0h | 0h | 1.5h | 0.5h | 0.5h | 1h | 3.5h |
| Saturday | Mini-project | 0h | 0h | 0h | 0h | 0h | 2h | 2h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **4h** | **3h** | **3.5h** | **4.5h** | **5h** | **26h** |

## By the end of this week you can…

- Turn a raw purchase-order table into a spend cube and name, with numbers, where the tail spend and maverick buying live.
- Build a weighted supplier scorecard from order-and-receipt data and explain why a perfect-looking score from two orders deserves suspicion, not celebration.
- Build a TCO model that shows why the cheapest unit price and the cheapest total cost are frequently two different suppliers — and quantify exactly how much a supplier's lead-time variability adds to your safety stock bill.
- Recommend and defend a sourcing action — consolidate, dual-source, or reallocate award — with a should-cost model, not just a spreadsheet of unit prices.

## Up next

Week 7 — Demand Forecasting Foundations, where the same discipline you just applied to *supply-side* uncertainty (supplier lead time, defect rate) turns to *demand-side* uncertainty.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
