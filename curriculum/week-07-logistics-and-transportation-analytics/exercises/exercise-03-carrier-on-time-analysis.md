# Exercise 3 — Carrier On-Time Analysis

**Goal:** Build the full carrier scorecard from Lecture 3 — on-time percentage, total spend, and cost per pound, per carrier — and correctly flag which carriers you don't have enough data on yet to trust.

**Estimated time:** 60 minutes.

## Setup

```sql
SELECT COUNT(*) FROM shipments;   -- must print 124
```

Create `solutions.sql` and put each answer under a `-- Task N` comment.

## Tasks

1. **On-time flag, row by row.** Write a `SELECT` returning `shipment_id`, `carrier`, `promised_transit_days`, `actual_transit_days`, and a computed `on_time` column (1 or 0) using `CASE WHEN actual_transit_days <= promised_transit_days THEN 1 ELSE 0 END`. *(Expected: 124 rows.)*

2. **On-time percentage per carrier.** Group Task 1 by `carrier`, computing `COUNT(*)` and on-time percentage, sorted descending by percentage. *(Spot check: `Yellowline Freight` should show **100.0%** on **16** shipments.)*

3. **Total spend per carrier.** For each `carrier`, compute `SUM(freight_cost)`, sorted descending. Which carrier has the highest total spend? Is that carrier necessarily the "most expensive" one — why or why not? *(Hint: revisit Lecture 3, Section 2.)*

4. **The combined scorecard.** Write one query returning, per carrier: `n_shipments`, `on_time_pct`, `total_spend`, and `avg_cost_per_lb` (computed as `AVG(freight_cost / weight_lbs)`), sorted by `on_time_pct` descending, then `avg_cost_per_lb` ascending.

5. **Filter out low-confidence carriers.** Re-run Task 4 with a `HAVING COUNT(*) >= 10` clause. Which carrier(s) disappear from the ranked list? List them by name and their (small) shipment count. *(Expected: `Pacific Rim Ocean Lines` and `SkyBridge Air Cargo` should drop out — both run only 3 shipments in this dataset.)*

6. **Mode-level reliability.** Instead of grouping by `carrier`, group by `mode` and compute on-time percentage for each. Which mode is the *least* reliable overall, and does that surprise you given what you know about each mode's typical distance and complexity?

7. **A carrier/mode cross-tab.** For carriers that operate in **more than one mode** (hint: you'll need a subquery or `HAVING COUNT(DISTINCT mode) > 1`), show their on-time percentage broken out separately by mode. Is any carrier meaningfully more reliable in one mode than another?

## Expected result (spot checks)

- Task 1 → 124 rows, each with a 0/1 `on_time` flag.
- Task 2 → `Yellowline Freight`: 16 shipments, 100.0% on-time.
- Task 5 → exactly 2 carriers drop out under `HAVING COUNT(*) >= 10`: `Pacific Rim Ocean Lines` (3 shipments) and `SkyBridge Air Cargo` (3 shipments).

## Done when…

- [ ] `solutions.sql` has all 7 queries under `-- Task N` comments.
- [ ] Task 2's `Yellowline Freight` figure matches the spot check.
- [ ] Task 5 correctly identifies both low-volume carriers and you can explain, in one sentence, why dropping them from the *ranking* (not from the *data*) is the right move.
- [ ] You can state, from Task 6, which mode has the worst on-time percentage and offer one plausible operational reason why (weather exposure, longer transit giving more opportunities to slip, handoffs between legs, etc.).

## Stretch

- Compute a single "carrier value score" as `on_time_pct - (avg_cost_per_lb * 100)` (an arbitrary but defensible way to combine the two signals into one sortable number) and re-rank the qualifying carriers by it. Do you get a different #1 than ranking by on-time percentage alone?
- For the two low-volume carriers dropped in Task 5, write one sentence each on what you'd want to see (more shipments? a specific number?) before you'd be comfortable making a keep/drop call on them.

## Submission

Commit `solutions.sql` to your portfolio under `c40-week-07/exercise-03/`.
