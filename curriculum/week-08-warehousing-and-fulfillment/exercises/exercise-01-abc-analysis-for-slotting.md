# Exercise 1 — ABC Analysis for Slotting

**Goal:** Turn Lecture 2's window-function pattern into a query and a pandas script that classify all 20 Austin East SKUs into A/B/C velocity tiers, purely from the real `pick_lines` profile — no eyeballing, no manual sorting.

**Estimated time:** 1.5 hours.

## Setup

Confirm the seed tables are loaded:

```sql
SELECT COUNT(*) FROM warehouse_skus;  -- must print 20
SELECT COUNT(*) FROM pick_lines;      -- must print 225
```

## Tasks

### Part A — SQL

1. **Pick frequency.** Write a query that returns `sku_id` and `pick_line_count` (a straight `COUNT(*)` from `pick_lines`, grouped by `sku_id`), ordered by `pick_line_count` descending.

2. **Join in names.** Extend it to include `sku_name` and `category` from `warehouse_skus`, so the output is readable.

3. **Cumulative share.** Add `pct_of_total` and `cumulative_pct` columns using the running-total window-function pattern from Lecture 2, Section 3. Round both to 2 decimals.

4. **Classify.** Add an `abc_class` column using a `CASE` expression: `'A'` where `cumulative_pct <= 80`, `'B'` where `cumulative_pct <= 95`, `'C'` otherwise.

5. **Class summary.** Write a second query, `GROUP BY abc_class`, that reports for each class: the count of SKUs in it, the total pick-line count, and that total's percentage of the grand total. Confirm the three percentages sum to 100%.

### Part B — pandas

6. Reproduce Tasks 1–4 in pandas: `groupby("sku_id").size()`, sort descending, `cumsum()` for the running total, and `pd.cut()` or a manual `np.select()` for the A/B/C assignment. Confirm your SQL and pandas outputs agree on every SKU's class.

7. **A different metric.** Everything above classified by **pick-line frequency** (how many separate order lines called for a SKU). Now classify by **units picked** instead — `SUM(qty_picked)` grouped by `sku_id`, same cumulative-share logic. Does every SKU land in the same class under both metrics, or does at least one SKU move classes depending on which metric you use? Report the SKU(s) that move, and explain in one sentence *why* a SKU could be popular by line-count but not by units (or vice versa) — think about what a "line" vs. a "unit" actually represents for a customer's order.

## Expected results (spot checks)

- SKU 1 (`Hydration Bladder 2L`) is the highest-frequency SKU: `pick_line_count = 55`, `cumulative_pct ≈ 24.44`.
- The A/B cutoff (crossing 80% cumulative) falls **between SKU 8 and SKU 9** by pick-line frequency — Class A should contain exactly **8 SKUs**.
- Class C (by pick-line frequency) should contain exactly **5 SKUs**, all with `pick_line_count <= 3`.

## Done when…

- [ ] Every one of the 20 SKUs has a `pick_line_count`, `cumulative_pct`, and `abc_class` in both SQL and pandas, and the two agree.
- [ ] The class summary's three percentages sum to (approximately) 100%.
- [ ] You've computed the units-based classification and reported which SKU(s), if any, change class versus the line-frequency classification.

## Stretch

- Real ABC analysis sometimes uses a **weighted metric** instead of pure frequency or pure units — e.g., `pick_line_count × unit_cost` (a proxy for "how much handling-adjusted value does this SKU represent"). Compute that weighted score for all 20 SKUs and re-classify. Does the `4-Season Mountaineering Tent` (SKU 20, one pick, but the single most expensive item in the catalog at $380/unit) move up a class under the weighted metric? What does that suggest about when frequency-only ABC analysis under- or over-values a SKU?

## Submission

Commit `exercise-01.sql` and `exercise-01.py` to your portfolio under `c40-week-08/exercise-01/`.
