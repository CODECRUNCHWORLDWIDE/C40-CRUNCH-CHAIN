# Exercise 3 — Price-Variance Analysis

**Time:** ~1 hour. **Builds on:** Lecture 1 (cost thinking) and Lecture 3 (TCO's cost discipline).

Every category at Crunch Gear has a **standard cost** — the unit price locked in when a supplier contract was signed at the start of the quarter (January 1, 2026). What suppliers actually charge on each PO can drift from that standard: a fabric mill raises prices mid-quarter, a CMT shop holds the line, another quietly discounts. **Price-variance analysis** measures that drift in dollars, and tells you which suppliers to bring back to the negotiating table.

## Set up the standard-cost table

```sql
CREATE TABLE standard_costs (
    supplier        TEXT    NOT NULL,
    category        TEXT    NOT NULL,
    standard_price  NUMERIC NOT NULL,
    PRIMARY KEY (supplier, category)
);

INSERT INTO standard_costs VALUES
('Alpine Weave Mills',    'Fabric', 18.00),
('Everest Textile Co.',   'Fabric', 18.00),
('Highland Fabric Group', 'Fabric', 19.50),
('Andes Stitch Works',    'CMT',    22.00),
('Pacific Rim Garments',  'CMT',    21.00),
('Northstar Apparel Mfg', 'CMT',    15.00);
```

## Task 1 — Variance per PO line

Join `purchase_orders` to `standard_costs` and compute, per line: the price variance per unit (`actual − standard`) and the total dollar variance for that line (`variance_per_unit × qty_ordered`). A **positive** variance is **unfavorable** (you paid more than standard); a **negative** variance is **favorable** (you paid less).

```sql
SELECT
    po.po_id,
    po.supplier,
    po.order_date,
    po.unit_price          AS actual_price,
    sc.standard_price,
    ROUND(po.unit_price - sc.standard_price, 3)                   AS variance_per_unit,
    ROUND((po.unit_price - sc.standard_price) * po.qty_ordered, 2) AS variance_total
FROM purchase_orders po
JOIN standard_costs sc
    ON po.supplier = sc.supplier AND po.category = sc.category
ORDER BY po.supplier, po.order_date;
```

**Expected:** 15 rows (only Fabric and CMT lines have a standard cost defined — Trims, Packaging, and MRO/Indirect correctly drop out of the `JOIN`, since we never set a standard for them). Check `po_id = 4` (Alpine Weave Mills, March order): standard is $18.00, actual is $18.50, so `variance_per_unit` should be **0.500** and, at 5,200 units ordered, `variance_total` should be **2600.00** — a single unfavorable PO line worth $2,600.

## Task 2 — Roll up variance by supplier

Aggregate Task 1 to one row per supplier: total quantity, total dollar variance, and variance as a percent of that supplier's standard-cost spend.

```sql
SELECT
    po.supplier,
    SUM(po.qty_ordered) AS total_qty,
    ROUND(SUM((po.unit_price - sc.standard_price) * po.qty_ordered), 2) AS variance_total,
    ROUND(100.0 * SUM((po.unit_price - sc.standard_price) * po.qty_ordered)
          / SUM(sc.standard_price * po.qty_ordered), 2) AS variance_pct_of_standard
FROM purchase_orders po
JOIN standard_costs sc
    ON po.supplier = sc.supplier AND po.category = sc.category
GROUP BY po.supplier
ORDER BY variance_total DESC;
```

**Expected:**

| Supplier | Total qty | Variance total | Variance % of standard |
|---|---:|---:|---:|
| Alpine Weave Mills | 19,200 | **$5,400.00** (unfavorable) | +1.56% |
| Andes Stitch Works | 4,500 | **$2,080.00** (unfavorable) | +2.10% |
| Highland Fabric Group | 2,600 | **$1,040.00** (unfavorable) | +2.05% |
| Northstar Apparel Mfg | 1,850 | **$370.00** (unfavorable) | +1.33% |
| Pacific Rim Garments | 3,750 | **−$500.00** (favorable) | −0.63% |
| Everest Textile Co. | 8,900 | **−$2,940.00** (favorable) | −1.83% |

## Task 3 — The grand total, and what it means

Sum `variance_total` across all six suppliers.

**Expected: $5,450.00 net unfavorable.** Crunch Gear paid $5,450 more than standard cost this quarter, across its Fabric and CMT categories combined. Note this is a **net** figure — it's hiding real movement in both directions: Alpine alone drifted $5,400 unfavorable (their price crept from $18.00 standard to $18.20, then $18.50 by March — a real, trackable escalation you could raise in the next contract renewal), while Everest ran $2,940 *favorable* the whole quarter, quietly saving money nobody had to ask for. **Never report only the net variance** — it can mask a large unfavorable supplier and a large favorable one canceling out, when the business questions ("why is Alpine drifting?" and "can we lock in Everest's pricing?") are completely different and both worth asking.

## Task 4 — Trend the worst offender

Alpine Weave Mills is the largest unfavorable variance. Write a query (or a pandas `groupby`) that shows Alpine's `unit_price` by `order_date`, in order, to confirm whether the drift is a one-time jump or a gradual creep.

```sql
SELECT order_date, unit_price
FROM purchase_orders
WHERE supplier = 'Alpine Weave Mills'
ORDER BY order_date;
```

**Expected:** three POs at $18.20, one at $18.50 (the March 1 order) — a **single step change**, not a gradual creep. That distinction matters for the negotiation conversation: a step change usually has an identifiable cause (a raw-material cost pass-through, a fuel surcharge, a contract-anniversary increase) that's worth asking about directly, rather than a vague "prices are drifting up" conversation.

## Watch for

- **Joining on the wrong key silently drops or duplicates rows.** The `JOIN` here uses both `supplier` *and* `category` — if you joined on `supplier` alone and a supplier appeared in two categories with two different standard prices, you'd get a Cartesian blow-up (every PO line matching every standard-cost row for that supplier). None of this week's suppliers have that problem, but get in the habit of joining on the full natural key, not just the obvious one.
- **Sign convention.** This exercise defines positive variance as unfavorable (paid more than standard) — the opposite convention (positive = favorable/savings) is also common in finance. Whichever you use, state it explicitly in your output; an unlabeled variance number is a trap waiting for the next person who reads it.

When done, you've built all three tools this week's mini-project assembles into one recommendation: the spend cube (Exercise 1), the scorecard (Exercise 2), and now variance analysis (Exercise 3).
