# Exercise 1 — Compute Cost per Lane

**Goal:** Turn 124 raw shipment rows into the lane-level cost view a logistics analyst actually reports: total spend, cost per pound, and cost per unit shipped, per lane and per mode.

**Estimated time:** 60 minutes.

## Setup

Confirm the seed is loaded:

```sql
SELECT COUNT(*) FROM shipments;   -- must print 124
```

Create `solutions.sql` and put each answer under a `-- Task N` comment.

## Tasks

1. **Every lane, once.** Select the distinct `lane_id` values in the table, sorted alphabetically. *(Expected: 15 distinct lanes, including `HPH-ATX`.)*

2. **Shipment count and total spend per lane.** For each `lane_id`, compute `COUNT(*)` and `SUM(freight_cost)`, sorted by total spend descending. *(Spot check: `HPH-ATX` should be the single most expensive lane by total spend — despite having only 6 shipments — because it carries long-haul Ocean and Air freight.)*

3. **Cost per pound, per lane.** Extend Task 2 with `SUM(freight_cost) / SUM(weight_lbs)`, aliased `cost_per_lb`, rounded to 4 decimals. *(Spot check: `AUS-DAL` → **9 shipments**, total cost **$6,117.22**, cost per lb **≈ 0.1115**.)*

4. **Cost per unit shipped, per lane.** Same idea, but divide by `SUM(units_shipped)` instead of weight, aliased `cost_per_unit`. Which lane has the highest cost per unit? Which has the lowest? *(Hint: compare a Parcel-heavy lane to an FTL-heavy one.)*

5. **Mode mix within a lane.** For the `MEM-AUS` lane only, group by `mode` and show `COUNT(*)`, `SUM(weight_lbs)`, and `SUM(freight_cost)` for each mode present. *(Expected: three modes appear — `Parcel`, `FTL`, and `Intermodal`.)*

6. **The Hai Phong import lane.** Select every shipment on `HPH-ATX`, showing `mode`, `weight_lbs`, `freight_cost`, and `freight_cost / weight_lbs AS cost_per_lb`, sorted by mode. *(Expected: 6 rows — 3 `Ocean`, 3 `Air`. Confirm the Air rows run roughly 35-40x the cost-per-lb of the Ocean rows.)*

7. **Weight-break spot check.** On the `AUS-DEN` lane, find the single heaviest LTL shipment and the single lightest LTL shipment. Compute `freight_cost / weight_lbs` for each. Which one has the lower cost per pound, and does that match what Lecture 1 predicted about weight breaks?

8. **A cost-per-unit ranking across the whole network.** Across **all** shipments (not grouped by lane), compute `SUM(freight_cost) / SUM(units_shipped)` per `mode`, sorted descending. *(Expected order top to bottom: Air, Parcel, LTL, Ocean, FTL, Intermodal roughly — confirm your numbers land in that neighborhood; exact values depend on rounding.)*

## Expected result (spot checks)

- Task 1 → 15 distinct lanes.
- Task 3 → `AUS-DAL`: 9 shipments, $6,117.22 total, ≈0.1115 cost/lb.
- Task 5 → `MEM-AUS` shows exactly 3 modes.
- Task 6 → 6 rows total on `HPH-ATX` (3 Ocean, 3 Air).

## Done when…

- [ ] `solutions.sql` has all 8 queries under `-- Task N` comments.
- [ ] Task 3's `AUS-DAL` numbers match the spot check above.
- [ ] You can say, in one sentence, why Task 4's answer differs from Task 3's ranking (cost per pound and cost per unit don't have to agree — a lane full of light, dense units can rank differently on each).
- [ ] Task 8 shows Air as the most expensive mode per unit and Intermodal/FTL among the cheapest, matching the pattern from Lecture 1.

## Stretch

- Add a column to Task 3 for `AVG(promised_transit_days)` per lane, and eyeball whether the cheapest-per-pound lanes are also the slowest ones (they should be, mostly — that's the trade-off).
- Write a query that flags any lane where the **maximum** cost-per-lb shipment is more than **3x** the **minimum** cost-per-lb shipment on that same lane — a sign the lane is mixing very different shipment sizes and might benefit from consolidation (Lecture 1, Section 4).

## Submission

Commit `solutions.sql` to your portfolio under `c40-week-07/exercise-01/`.
