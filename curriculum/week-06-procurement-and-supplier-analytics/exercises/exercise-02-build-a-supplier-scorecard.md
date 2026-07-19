# Exercise 2 — Build a Supplier Scorecard

**Time:** ~1.5 hours. **Builds on:** Lecture 2 — Supplier Performance & Scoring.

Lecture 2 scored the three **CMT** suppliers. This exercise repeats the exact same method on a different category: the three **Fabric** suppliers — **Alpine Weave Mills**, **Everest Textile Co.**, and **Highland Fabric Group**. Don't copy Lecture 2's numbers; compute your own from scratch. The point is to prove you can reproduce the method, not memorize the result.

## Task 1 — Compute the four raw metrics

In SQL and/or pandas, compute for each Fabric supplier:

1. Volume-weighted average unit price
2. Defect rate (%)
3. On-time delivery rate (%)
4. Lead-time standard deviation, in days

```sql
SELECT
    supplier,
    ROUND(SUM(qty_ordered * unit_price) / SUM(qty_ordered), 3) AS avg_unit_price,
    ROUND(100.0 * SUM(defect_qty) / SUM(qty_received), 3) AS defect_rate_pct,
    ROUND(100.0 * SUM(CASE WHEN receipt_date <= promised_date THEN 1 ELSE 0 END)
          / COUNT(*), 1) AS otd_pct,
    COUNT(*) AS n_orders
FROM purchase_orders
WHERE category = 'Fabric'
GROUP BY supplier
ORDER BY supplier;
```

Get lead-time standard deviation in pandas (SQLite has no built-in `STDDEV`):

```python
import pandas as pd

df = pd.read_sql("SELECT * FROM purchase_orders WHERE category = 'Fabric'", conn,
                  parse_dates=['order_date', 'promised_date', 'receipt_date'])
df['lead_time_days'] = (df['receipt_date'] - df['order_date']).dt.days
print(df.groupby('supplier')['lead_time_days'].std().round(3))
```

**Expected (rounded to 2 decimals):**

| Supplier | Avg unit price | Defect rate % | OTD % | Lead-time std | n orders |
|---|---:|---:|---:|---:|---:|
| Alpine Weave Mills | 18.28 | 0.42 | 50.0 | 2.63 | 4 |
| Everest Textile Co. | 17.67 | 2.13 | 33.3 | 4.16 | 3 |
| Highland Fabric Group | 19.90 | 0.50 | 100.0 | 0.00 | 2 |

## Task 2 — Normalize each metric to 0–100

Write a `norm_lower_better(series)` and `norm_higher_better(series)` function (Lecture 2, Section 3) and apply them to the four columns from Task 1. Remember: cost, defect rate, and lead-time std are all "lower is better"; OTD is "higher is better."

```python
def norm_lower_better(s):
    return 100 * (s.max() - s) / (s.max() - s.min())

def norm_higher_better(s):
    return 100 * (s - s.min()) / (s.max() - s.min())
```

**Expected:** Everest should score **0** on cost (it's the most expensive... check that against your Task 1 table before assuming — the cheapest unit price scores 100, not 0, so make sure you know which supplier is actually cheapest before checking your normalized scores).

## Task 3 — Weighted score

Apply Lecture 2's weights — 30% cost, 30% quality, 25% OTD, 15% lead-time reliability — and compute one weighted score per supplier.

**Expected (rounded to 1 decimal):**

| Supplier | Cost score | Quality score | OTD score | Lead-var score | **Weighted score** |
|---|---:|---:|---:|---:|---:|
| Alpine Weave Mills | 72.6 | 100.0 | 25.0 | 36.8 | **63.6** |
| Everest Textile Co. | 100.0 | 0.0 | 0.0 | 0.0 | **30.0** |
| Highland Fabric Group | 0.0 | 95.2 | 100.0 | 100.0 | **68.6** |

## Task 4 — Interpret, don't just report

In a short paragraph (in a comment block or a `notes.md`), answer:

1. Highland Fabric Group edges out Alpine Weave Mills for the top score, despite having the *worst* (most expensive) unit price in the group. Which components pulled Highland up, and by how much did the OTD gap alone matter?
2. Highland's scorecard is built from only **2 orders** — the same small-sample caution from Lecture 2, Section 5 applies here. Would you recommend shifting significant Fabric volume to Highland based on this scorecard alone? What would you want to see first?
3. Alpine Weave Mills is 62.6% of all Fabric spend (from Exercise 1) but only ranks 2nd of 3 on the scorecard. Is that a contradiction? What does it tell a category manager about where to focus a supplier-improvement conversation?

## Watch for

- **Direction errors are the most common bug here.** If your "cheapest" supplier scores 0 instead of 100 on cost, you used `norm_higher_better` where you needed `norm_lower_better` (or vice versa) — reread Lecture 2, Section 3 before debugging further.
- **A tie at the max or min of a normalized column isn't a bug.** If two suppliers were equally cheap, they'd both legitimately score 100 on cost — that's the min-max formula working correctly, not an error to "fix."

When done, keep your scorecard function — Challenge 1 this week reuses it (with a third supplier and capacity limits added) to make a real award-allocation decision.
