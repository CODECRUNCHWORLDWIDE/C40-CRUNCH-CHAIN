# Lecture 2 — Supplier Performance & Scoring

> **Duration:** ~2 hours. **Outcome:** You can compute the four component metrics of a supplier scorecard (cost, quality, on-time delivery, lead-time reliability) from raw order-and-receipt data, combine them into one weighted score, and explain the scorecard's real limitations — especially the small-sample trap.

Lecture 1 answered "where does the money go." This lecture answers the harder question: **is it going to the right suppliers?** A supplier who quotes the lowest unit price but ships late, ships short, and ships defective product isn't cheap — they're expensive in ways that don't show up on the purchase order. A **supplier scorecard** is the tool that makes those hidden costs visible and comparable, by scoring every supplier on the same weighted set of criteria from the same underlying data.

We work the three **CMT** (Cut-Make-Trim — the contract manufacturers who sew Crunch Gear's jackets) suppliers from this week's seed data for every example: **Andes Stitch Works**, **Pacific Rim Garments**, and **Northstar Apparel Mfg**.

## 1. The four components of a scorecard

Different companies weight scorecards differently, but almost every one built for a manufacturing or apparel supply chain covers some version of these four:

| Component | What it measures | Computed from |
|---|---|---|
| **Cost** | How competitive is the price? | `unit_price`, volume-weighted across orders |
| **Quality** | How much of what shipped was actually usable? | `defect_qty / qty_received` |
| **On-time delivery (OTD)** | Did it arrive when promised? | `receipt_date <= promised_date` |
| **Lead-time reliability** | How *consistent* is the lead time, order to order? | standard deviation of (`receipt_date - order_date`) |

Notice that OTD and lead-time reliability are related but **not the same measurement**. OTD asks "was this specific shipment late?" — a yes/no per order. Lead-time reliability asks "how much does this supplier's lead time *swing* from order to order?" — a supplier could hit their promise date 100% of the time by simply quoting a generous lead time every time (e.g., always promising 45 days when they could do 25), and still be operationally painful to plan around if the *actual* lead time bounces between 20 and 45 days unpredictably. You need both numbers; neither alone tells the full story.

## 2. Computing each component from the seed data

**Cost** — the volume-weighted average unit price actually paid (not a simple average of PO-line prices, which would overweight small orders):

```sql
SELECT
    supplier,
    ROUND(SUM(qty_ordered * unit_price) / SUM(qty_ordered), 3) AS avg_unit_price
FROM purchase_orders
WHERE category = 'CMT'
GROUP BY supplier;
```

**Quality** — total defective units received divided by total units received (not averaged per-order, for the same volume-weighting reason):

```sql
SELECT
    supplier,
    ROUND(100.0 * SUM(defect_qty) / SUM(qty_received), 2) AS defect_rate_pct
FROM purchase_orders
WHERE category = 'CMT'
GROUP BY supplier;
```

**On-time delivery** — the percent of orders where receipt happened at or before the promised date:

```sql
SELECT
    supplier,
    ROUND(100.0 * SUM(CASE WHEN receipt_date <= promised_date THEN 1 ELSE 0 END)
          / COUNT(*), 1) AS otd_pct
FROM purchase_orders
WHERE category = 'CMT'
GROUP BY supplier;
```

**Lead-time reliability** — the standard deviation, in days, of actual lead time (`receipt_date - order_date`) across each supplier's orders:

```sql
SELECT
    supplier,
    ROUND(STDDEV_SAMP(receipt_date - order_date), 2) AS lead_time_std_days
    -- SQLite has no built-in STDDEV; compute it in Python/pandas instead (see below)
FROM purchase_orders
WHERE category = 'CMT'
GROUP BY supplier;
```

`STDDEV_SAMP` is PostgreSQL's sample standard deviation aggregate. SQLite doesn't ship one built in — this is a good moment to reach for pandas instead, which is exactly how a real analyst would do it:

```python
import pandas as pd

df = pd.read_sql("SELECT * FROM purchase_orders WHERE category = 'CMT'", conn,
                  parse_dates=['order_date', 'promised_date', 'receipt_date'])
df['lead_time_days'] = (df['receipt_date'] - df['order_date']).dt.days

lead_time_std = df.groupby('supplier')['lead_time_days'].std()
print(lead_time_std.round(2))
```

Running all four against the seed data gives:

| Supplier | Avg unit price | Defect rate | OTD % | Lead-time std (days) |
|---|---:|---:|---:|---:|
| Andes Stitch Works | $22.462 | 0.92% | 33.3% | 2.31 |
| Pacific Rim Garments | $20.867 | 3.51% | 33.3% | 3.79 |
| Northstar Apparel Mfg | $15.200 | 0.76% | 100.0% | 0.00 |

Already, before any weighting, a picture emerges: Northstar is cheaper, cleaner, and perfectly on-time — but keep reading before you conclude Northstar is simply "the best supplier." Section 4 explains why.

## 3. Turning four different units into one comparable score

You can't average a dollar figure, a percentage, and a day-count directly — they're not on the same scale, and for cost/quality/lead-time-std, *lower* is better, while for OTD, *higher* is better. The standard fix is **min-max normalization**: rescale each metric to a common 0–100 scale within the comparison group, flipping direction where needed so that 100 always means "best in the group."

For a metric where **lower is better** (cost, defect rate, lead-time std):

```
score = 100 × (worst_value − this_value) / (worst_value − best_value)
```

For a metric where **higher is better** (OTD):

```
score = 100 × (this_value − worst_value) / (best_value − worst_value)
```

In pandas:

```python
def norm_lower_better(s):
    return 100 * (s.max() - s) / (s.max() - s.min())

def norm_higher_better(s):
    return 100 * (s - s.min()) / (s.max() - s.min())

scorecard['cost_score']    = norm_lower_better(scorecard['avg_unit_price'])
scorecard['quality_score'] = norm_lower_better(scorecard['defect_rate'])
scorecard['otd_score']     = norm_higher_better(scorecard['otd_pct'])
scorecard['leadvar_score'] = norm_lower_better(scorecard['lead_time_std'])
```

Now every component lives on the same 0–100 scale, and 0 always means "worst in this comparison group" while 100 always means "best." **Important caveat:** min-max normalization is *relative*, not absolute — a supplier scoring 100 on quality only means they had the lowest defect rate *among the suppliers you compared*, not that they hit some universal quality bar. Add a fourth supplier with an even better defect rate next quarter, and everyone else's quality score shifts, even though their actual defect rate didn't change. Know this before you present a scorecard as if the numbers were absolute truth.

## 4. Weighting and combining into one score

Pick weights that reflect what the business actually cares about most. A reasonable default for an apparel CMT relationship, where quality escapes are costly (returned goods, brand damage) and cost still matters a great deal in a thin-margin category:

```
weighted_score = 0.30 × cost_score + 0.30 × quality_score + 0.25 × otd_score + 0.15 × leadvar_score
```

```python
weights = dict(cost=0.30, quality=0.30, otd=0.25, leadvar=0.15)
scorecard['weighted_score'] = (
    scorecard['cost_score']    * weights['cost'] +
    scorecard['quality_score'] * weights['quality'] +
    scorecard['otd_score']     * weights['otd'] +
    scorecard['leadvar_score'] * weights['leadvar']
)
```

Running this on the three CMT suppliers:

| Supplier | Cost score | Quality score | OTD score | Lead-var score | **Weighted score** |
|---|---:|---:|---:|---:|---:|
| Andes Stitch Works | 0.0 | 94.1 | 0.0 | 39.0 | **34.1** |
| Northstar Apparel Mfg | 100.0 | 100.0 | 100.0 | 100.0 | **100.0** |
| Pacific Rim Garments | 22.0 | 0.0 | 0.0 | 0.0 | **6.6** |

Northstar Apparel Mfg tops every single component — cheapest, cleanest, always on time, zero lead-time variance — and the weighted score reflects that with a clean sweep. Andes comes in a distant second (excellent quality keeps them afloat, but the highest price and worst-tied OTD hurt), and Pacific Rim finishes last, dragged down by the worst defect rate and worst lead-time variability in the group on top of a below-Andes-but-not-cheap price.

## 5. The trap this scorecard is hiding: sample size

Look back at the raw data one more time. Northstar Apparel Mfg's perfect score is built from **two purchase orders**. Andes and Pacific Rim each contributed **three**. A supplier who ships two-for-two on time isn't necessarily more reliable than one who ships eight-for-ten — they might just not have had a bad day *yet*. Two data points can't distinguish "genuinely excellent process" from "got lucky twice." This is the single most common mistake in real-world scorecarding: treating every supplier's score as equally statistically confident, when the underlying sample sizes might differ by 5–10x.

Two practical fixes, both worth knowing:

- **Report the sample size alongside the score.** A scorecard column showing `n_orders` next to `weighted_score` costs nothing and instantly tells a reader how much to trust a 100.0.
- **Widen the observation window before acting on extremes.** A supplier who scores 100 or 0 after 2–3 orders should be watched for another quarter before you make a sourcing decision that assumes the score is stable — this week's Challenge 1 (Supplier Award Allocation) puts you in exactly this position.

This doesn't mean the scorecard is wrong to compute — Northstar genuinely performed better on every order it shipped this quarter. It means you, the analyst, must attach the right amount of confidence to that number before a VP reads "100.0" and assumes it's guaranteed to repeat at 5x the volume.

## 6. Doing it entirely in SQL, in one query

Everything above can be done as one CTE chain — useful once you're comfortable with the pieces individually:

```sql
WITH metrics AS (
    SELECT
        supplier,
        SUM(qty_ordered * unit_price) / SUM(qty_ordered)              AS avg_unit_price,
        100.0 * SUM(defect_qty) / SUM(qty_received)                    AS defect_rate,
        100.0 * SUM(CASE WHEN receipt_date <= promised_date THEN 1 ELSE 0 END) / COUNT(*) AS otd_pct,
        COUNT(*)                                                       AS n_orders
    FROM purchase_orders
    WHERE category = 'CMT'
    GROUP BY supplier
)
SELECT
    supplier,
    ROUND(avg_unit_price, 3) AS avg_unit_price,
    ROUND(defect_rate, 2)    AS defect_rate_pct,
    ROUND(otd_pct, 1)        AS otd_pct,
    n_orders
FROM metrics
ORDER BY avg_unit_price;
```

(Lead-time standard deviation still needs pandas, per Section 2 — a good example of a case where SQL and Python genuinely complement each other rather than compete: SQL for the set-based aggregation, pandas for the statistical function SQLite doesn't ship.)

## 7. Check yourself

- Why can't you average a dollar price, a percentage, and a day-count directly into one score?
- What does min-max normalization actually do, and why does it flip direction for "lower is better" metrics?
- Northstar Apparel Mfg scored a perfect 100.0 this quarter. Name two reasons that number deserves more scrutiny before you triple their order volume.
- Why are OTD and lead-time reliability measuring genuinely different things, even though they're both "about delivery timing"?
- If you changed the scorecard weights to 50% cost / 20% quality / 20% OTD / 10% lead-time, would Pacific Rim's rank relative to Andes change? (Work it out — you have the component scores above.)

If those are automatic, Lecture 3 goes one level deeper: even a *fair* scorecard only tells you about price, quality, and delivery *as observed* — it doesn't yet tell you what a supplier's unreliability is quietly costing you in extra inventory. That's total cost of ownership.

## Further reading

- **APICS/ASCM — Supplier Scorecard fundamentals** (search "supplier scorecard" in the SCM Dictionary): <https://www.ascm.org/learning-development/certifications-credentials/scmdictionary/> — the industry-standard vocabulary this lecture follows.
- **PostgreSQL — Aggregate Functions, including `STDDEV_SAMP`:** <https://www.postgresql.org/docs/current/functions-aggregate.html>
- **pandas — `groupby` and `.std()`:** <https://pandas.pydata.org/docs/reference/api/pandas.core.groupby.GroupBy.std.html>
- **Investopedia — "Normalization":** <https://www.investopedia.com/terms/n/normalization.asp> — a finance-adjacent explanation of min-max scaling, the same technique used here for scorecards.
