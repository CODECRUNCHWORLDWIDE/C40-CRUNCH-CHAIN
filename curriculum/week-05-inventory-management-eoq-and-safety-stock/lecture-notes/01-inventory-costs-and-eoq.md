# Lecture 1 — Inventory Costs & EOQ

> **Duration:** ~2 hours. **Outcome:** you can name and compute the three inventory cost buckets, derive the EOQ formula from a total-cost equation instead of memorizing it, compute EOQ for a real SKU in SQL and pandas, and explain why the total-cost curve barely moves near its own optimum.

Every unit Crunch Gear keeps on a warehouse shelf is a decision with a cost attached, whether anyone wrote that cost down or not. Order too much at once and you tie up cash and pay to store it. Order too little, too often, and you pay an army of small fees to place, receive, and expedite orders. Order confidently and get surprised by demand, and you either turn away a paying customer or scramble a rush shipment. This lecture is about naming those costs precisely enough to add them up, and then choosing an order quantity that makes the sum as small as it can be.

## 1. The three cost buckets

Every unit of inventory sitting in a warehouse, or *not* sitting there when a customer wants it, generates one of three kinds of cost.

### Holding cost (carrying cost)

The cost of owning inventory *while it sits there*, for every day it sits there. It's rarely one line item — it's several, bundled:

- **Cost of capital.** Money spent on inventory isn't invested in something else. If Crunch Gear's cost of capital is 8%, every dollar tied up in a jacket that could instead pay down debt or fund a new product line is costing 8¢/year in opportunity cost.
- **Storage.** Warehouse rent, utilities, racking, and the labor to move product in and out.
- **Insurance and taxes.** Inventory is an insured, taxed asset in most jurisdictions.
- **Obsolescence and shrinkage.** Fashion risk (a jacket color goes out of style), damage, theft, expiry.

Rather than track each of these separately, operations teams almost always collapse holding cost into a single **annual holding rate**, `i`, expressed as a fraction of the unit's cost, and compute:

```
H = i * C
```

where `C` is unit cost and `H` is the dollar holding cost **per unit, per year**. A typical annual holding rate in apparel and general merchandise runs **18–30%** — Crunch Gear's finance team charges Jackets 25% (higher fashion/obsolescence risk, higher per-unit capital) and Accessories 18–20% (cheap, durable, low fashion risk). This is exactly what you saw in this week's seed data — `holding_pct` is not one constant, and it shouldn't be.

### Ordering cost (setup cost)

The fixed cost of placing **one purchase order**, regardless of how many units are on it: purchasing-team time, the supplier's minimum-order administration, receiving and inspection labor, and any fixed portion of inbound freight (a partial truck costs almost as much to schedule as a full one). Crunch Gear budgets **`S` = $60–$120 per PO**, depending on the supplier and category (Accessories suppliers are simpler to order from than the Jackets contract manufacturer, hence the lower `order_cost` for SKUs 8 and 10 in the seed data).

Ordering cost is why you don't just order one unit at a time the moment you need it — every order carries this fixed tax, so spreading it across a bigger batch dilutes its per-unit impact.

### Shortage cost (stockout cost)

The cost of *not having* a unit when demand shows up: a lost sale and its margin, a backorder and the cost of expediting it, or — the hardest to put a number on — the goodwill and reputation damage of an empty shelf or a "sorry, backordered" email. Shortage cost is what safety stock exists to prevent, and it's the subject of Lecture 2. For this lecture, we assume shortages don't happen (the classic EOQ assumption) so we can isolate the trade-off between holding and ordering cost first.

## 2. Setting up the total-cost equation

Picture ordering the **same fixed quantity `Q`** every time, at a **constant, known annual demand `D`**. Inventory looks like a sawtooth: it starts at `Q` right after a delivery, drains linearly to zero at rate `D`, and gets refilled to `Q` again.

Two things follow directly from that sawtooth:

- **Average inventory on hand** is `Q/2` (halfway between the peak `Q` and the trough `0`). That's what you're paying holding cost on, all year.
- **Number of orders per year** is `D/Q`. That's how many times you pay the fixed ordering cost `S`.

So the two costs that trade off against each other, as a function of order quantity `Q`, are:

```
Annual holding cost  = (Q/2) * H
Annual ordering cost = (D/Q) * S
```

**Total annual cost** (excluding the cost of the goods themselves — `D * C` is fixed regardless of `Q`, so it never affects the optimal order size and is usually left out of this equation):

```
TC(Q) = (D/Q) * S  +  (Q/2) * H
```

Look at what each term does as `Q` grows:

- Order **bigger** batches (`Q` up) → **fewer** orders/year → ordering cost **falls**, but average inventory **rises** → holding cost **rises**.
- Order **smaller** batches (`Q` down) → holding cost **falls**, but ordering cost **rises**.

Somewhere in between is a `Q` that minimizes the sum. That's the Economic Order Quantity.

## 3. Deriving EOQ

Minimize `TC(Q)` with calculus: take the derivative with respect to `Q`, set it to zero.

```
TC(Q) = D*S*Q^(-1) + (H/2)*Q

dTC/dQ = -D*S*Q^(-2) + H/2
```

Set `dTC/dQ = 0` and solve for `Q`:

```
H/2 = D*S / Q^2
Q^2 = 2*D*S / H
Q* = sqrt(2*D*S / H)
```

That's the **EOQ formula**:

```
EOQ = sqrt( 2 * D * S / H )
```

You don't need to memorize a derivation you can't reconstruct — you need to be able to reconstruct it. If you forget the formula on the job, you can always re-derive it from `TC(Q) = (D/Q)S + (Q/2)H` in under a minute, and that habit will save you the day you need EOQ under a slightly different cost structure the formula sheet doesn't cover.

**A useful property, free from the derivation:** at `Q = Q*`, the two cost terms are *equal*. Holding cost at the optimum exactly equals ordering cost at the optimum. That's a fast sanity check on any EOQ calculation — if your holding-cost-at-Q\* and ordering-cost-at-Q\* aren't equal, you made an arithmetic error somewhere.

## 4. Worked example — Alpine Shell Jacket

From this week's seed catalog, SKU 1:

| Input | Value |
|---|---|
| Annual demand `D` | 4,800 units/year |
| Order cost `S` | $120/order |
| Unit cost `C` | $84.00 |
| Holding rate `i` | 0.25 (25%/year) |

First, holding cost per unit per year:

```
H = i * C = 0.25 * 84.00 = $21.00/unit/year
```

Then EOQ:

```
Q* = sqrt(2 * 4800 * 120 / 21) = sqrt(1,152,000 / 21) = sqrt(54,857.1) ≈ 234.2 units
```

Round to a sensible pack size — **234 units**. Two things follow immediately:

```
Orders per year = D / Q* = 4800 / 234.2 ≈ 20.5 orders/year
Cycle time       = 365 / 20.5 ≈ 17.8 days between orders
```

And the minimum total cost:

```
TC(Q*) = (D/Q*)*S + (Q*/2)*H
       = (4800/234.2)*120 + (234.2/2)*21
       = 2,460.0 + 2,459.1
       ≈ $4,919/year
```

Notice the two terms came out almost identical (~$2,460 each) — that's the "holding cost equals ordering cost at the optimum" property, confirming the arithmetic. There's also a shortcut formula for the minimum cost itself, useful for a quick check without computing `Q*` first:

```
TC(Q*) = sqrt(2 * D * S * H) = sqrt(2 * 4800 * 120 * 21) = sqrt(24,192,000) ≈ $4,918.5/year
```

(matches, modulo rounding).

## 5. Computing EOQ for a whole catalog — SQL and pandas

For one SKU, a calculator is fine. For a catalog, you want a query or a pandas expression, because square roots inside `SELECT` and `.apply()` are exactly what these tools are for.

**SQL (PostgreSQL — `sqrt()` is built in; SQLite needs `sqrt()` too, available since 3.35 or via `pow(x, 0.5)`):**

```sql
SELECT
    sku_id,
    sku_name,
    annual_demand                      AS d,
    order_cost                         AS s,
    ROUND(unit_cost * holding_pct, 2)  AS h,
    ROUND(
        SQRT(2 * annual_demand * order_cost / (unit_cost * holding_pct)),
        1
    )                                  AS eoq
FROM skus
ORDER BY sku_id;
```

**pandas (equivalent, and where you'll do the rest of this week's heavier lifting):**

```python
import pandas as pd
import numpy as np
from sqlalchemy import create_engine

engine = create_engine("postgresql://localhost/crunch_chain_wk5")
skus = pd.read_sql("SELECT * FROM skus", engine)

skus["h"] = skus["unit_cost"] * skus["holding_pct"]
skus["eoq"] = np.sqrt(2 * skus["annual_demand"] * skus["order_cost"] / skus["h"])
skus["orders_per_year"] = skus["annual_demand"] / skus["eoq"]
skus["cycle_days"] = 365 / skus["orders_per_year"]
skus["tc_at_eoq"] = np.sqrt(2 * skus["annual_demand"] * skus["order_cost"] * skus["h"])

print(skus[["sku_name", "eoq", "orders_per_year", "cycle_days", "tc_at_eoq"]].round(1))
```

Run this against the full catalog and notice the pattern: high-volume, cheap SKUs (Insulated Water Bottle, SKU 8: `D`=12,000, `C`=$18) get a *large* EOQ — you order a lot at once because each unit is cheap to hold. Low-volume, expensive SKUs (Summit Down Parka, SKU 2: `D`=1,500, `C`=$165) get a *small* EOQ relative to their demand — expensive holding cost per unit pushes you toward smaller, more frequent orders even though the unit is ordered less often overall.

## 6. Total-cost sensitivity — why EOQ has a "flat bottom"

Here's a property of the EOQ model worth understanding deeply, because it changes how you should behave in the real world: **`TC(Q)` is very forgiving of getting `Q` wrong.**

There's a clean formula for how much extra cost you pay if you order some quantity `Q` instead of the true optimum `Q*`. Define the ratio `r = Q / Q*`. Then:

```
TC(Q) / TC(Q*) = (1/2) * (r + 1/r)
```

Plug in some ratios:

| You order... | `r` | `TC(Q)/TC(Q*)` | Cost penalty |
|---|---|---|---|
| Exactly `Q*` | 1.00 | 1.000 | 0% |
| 20% too much | 1.20 | 1.017 | **1.7%** |
| 20% too little | 0.80 | 1.025 | **2.5%** |
| 50% too much | 1.50 | 1.083 | **8.3%** |
| Half of `Q*` | 0.50 | 1.250 | **25%** |
| Double `Q*` | 2.00 | 1.250 | **25%** |

Being 20% off the true EOQ costs you under 2% in extra total cost. This is *not* an excuse to ignore EOQ — it's the reason EOQ is such a durable, widely used tool despite every one of its assumptions being technically false in the real world (see below). You don't need `S`, `H`, and `D` measured to three decimal places to get most of the benefit; a rough estimate gets you within a couple of percent of optimal, every time. What you *cannot* forgive is ignoring the trade-off entirely — ordering one unit at a time, or ordering a year's supply at once, both land you in the expensive tails of that curve.

## 7. The assumptions EOQ makes — and when they break

The classic EOQ model assumes:

1. **Demand is constant and known** — no seasonality, no forecast error.
2. **Lead time is zero or constant** — the order arrives instantly, or after a fixed, certain delay.
3. **No stockouts** — every unit of demand is met.
4. **Unit cost is fixed** — no quantity discounts for ordering more.
5. **Infinite, continuous planning horizon** — no end-of-life, no minimum/maximum order constraints.
6. **One item at a time** — no interaction between SKUs sharing a truck or a supplier minimum.

Every one of these is false in Crunch Gear's actual warehouse. Demand has seasonality (jackets sell more in fall) and forecast error (Weeks 3–4). Lead time varies (Lecture 2 handles this explicitly). Quantity discounts are common (order 500+ units, pay 5% less per unit — solved by comparing `TC(Q)` at each discount breakpoint, not covered further here). None of that makes EOQ useless — it makes EOQ the **starting point**, not the final answer. You compute the deterministic EOQ first because it's a clean, defensible baseline; then Lecture 2 layers safety stock on top of it to handle the uncertainty EOQ deliberately ignores.

## 8. Check yourself

- Name the three inventory cost buckets. Which one does the classic EOQ model assume away?
- Write `H` in terms of unit cost `C` and holding rate `i`. Why does a $165 parka have a higher `H` than an $18 water bottle even at the same holding rate?
- Derive `Q* = sqrt(2DS/H)` from `TC(Q) = (D/Q)S + (Q/2)H` without looking back at this lecture.
- At `Q = Q*`, what's true about the relationship between holding cost and ordering cost?
- If you order 50% more than the true EOQ, roughly what percentage of extra total cost do you pay? Why is this forgiving property important in practice?
- Name two EOQ assumptions that don't hold at Crunch Gear, and which later lecture/week addresses each one.

If those are automatic, Lecture 2 adds the piece EOQ leaves out on purpose: what happens when demand and lead time aren't perfectly known.

## Further reading

- **APICS/ASCM — Economic Order Quantity overview:** <https://www.ascm.org/>
- **PostgreSQL — Mathematical functions (`sqrt`, `power`):** <https://www.postgresql.org/docs/current/functions-math.html>
- **SQLite — Math functions:** <https://www.sqlite.org/lang_mathfunc.html>
- **NumPy — `numpy.sqrt`:** <https://numpy.org/doc/stable/reference/generated/numpy.sqrt.html>
