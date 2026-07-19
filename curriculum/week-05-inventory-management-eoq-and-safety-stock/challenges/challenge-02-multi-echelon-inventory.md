# Challenge 2 — Multi-Echelon Inventory Positioning

**Time:** ~2 hours. **Difficulty:** Hard.

## The scenario

Crunch Gear sells a new SKU, the **Crunch Trail Buff**, out of three regional warehouses: **Austin East**, **Denver Central**, and **Portland West**. Right now, each warehouse orders directly from the overseas supplier and carries its own safety stock, sized to its own local demand. The VP of Operations has floated an idea: consolidate all safety stock at a single **Central Distribution Center (CDC)** and let the three warehouses pull from it. Her pitch is the textbook one — "pooling demand across locations should let us carry less total safety stock for the same service level." Your job is to check whether that's actually true here, by how much, and what it costs to get the reasoning wrong.

## The data

```sql
CREATE TABLE warehouse_demand (
    warehouse       TEXT    PRIMARY KEY,
    daily_demand_mean NUMERIC NOT NULL,   -- units/day, Crunch Trail Buff
    daily_demand_std  NUMERIC NOT NULL    -- units/day
);

INSERT INTO warehouse_demand VALUES
('Austin East',     18.0, 6.0),
('Denver Central',  14.0, 5.5),
('Portland West',   11.0, 4.8);
```

| Parameter | Value |
|---|---|
| Supplier → CDC lead time `L` | 20 days |
| Supplier → CDC lead-time std dev `σ_L` | 3.0 days |
| Target cycle-service level | 95% (`z` = 1.645) |
| Unit cost | $12.00 |
| Annual holding rate | 20% (`H` = $2.40/unit/year) |

Assume warehouse-level daily demand is **independent** across the three locations (a fair simplifying assumption — these are geographically separate customer bases with no shared driver of day-to-day demand swings).

## Your task

### Part A — decentralized baseline

1. For each warehouse, compute `σ_DLT` and safety stock at the 95% target, using the standard formula from Lecture 2 (`σ_DLT = sqrt(L*σ_d² + d̄²*σ_L²)`, applying each warehouse's own `L` and `σ_L`, currently the full supplier lead time since each orders direct).
2. Sum the three safety-stock numbers. This is the total safety stock Crunch Gear carries today, decentralized.
3. Convert to an annual holding-cost dollar figure using `H = unit_cost * holding_pct`.

### Part B — the naive square-root law (and why it's tempting)

The classic "square-root law" of inventory pooling says: if you consolidate `n` independent locations into one, and only *demand* variability matters (not lead-time variability), total safety stock scales down by a factor of `sqrt(n)`.

4. Compute what the VP's pitch would predict using this shortcut: `SS_naive_centralized = SS_decentralized_total / sqrt(3)`. Convert to a dollar figure and compute the "savings" this shortcut promises.

### Part C — the correct centralized calculation

5. Now do it properly. At the CDC, safety stock must cover the **aggregated** demand across all three warehouses over the CDC's own supplier lead time:
   - Aggregate mean demand: `d_agg = sum of the three daily_demand_mean values`.
   - Aggregate demand std dev (pooling independent variances): `σ_d_agg = sqrt(sum of the three daily_demand_std² values)`.
   - `σ_DLT_agg = sqrt(L * σ_d_agg² + d_agg² * σ_L²)` — same formula shape as Part A, just with the aggregated numbers.
   - `SS_centralized = z * σ_DLT_agg`.
6. Convert to a dollar figure. Compare it honestly to **both** the decentralized total (Part A) and the naive square-root-law prediction (Part B).

### Part D — explain the gap

7. The naive law and the correct calculation should give noticeably different answers. Explain **why**, specifically: which term inside `σ_DLT_agg` — the demand-variability term or the lead-time-variability term — pools nicely when you aggregate across warehouses, and which one does *not*? *(Hint: mean demand aggregates by simple addition, `d_agg = Σd_i`. Variance aggregates by `Σσ_i²`. Look at which one the lead-time-variability term depends on, squared, and think about what happens to `(Σd_i)²` versus `Σ(d_i²)`.)*
8. State, in dollars, what it would have cost Crunch Gear to trust the naive square-root law instead of the correct calculation — specifically, how many fewer units of safety stock the naive law would have told them to carry at the CDC than they actually need to hit 95%. Is that a small rounding difference or a real exposure?

## Stretch — the transportation trade-off the pitch didn't mention

Centralizing doesn't just change *how much* safety stock you carry — it can add a leg to the network. If the CDC has to ship to each warehouse after receiving from the supplier, a warehouse's effective replenishment lead time is no longer just the supplier's 20 days; it's the supplier-to-CDC leg **plus** a CDC-to-warehouse transfer leg.

9. Suppose the CDC-to-warehouse transfer adds a mean of 4 days with a std dev of 1.0 day, independent of the supplier leg. The combined lead time is `L_total = 20 + 4 = 24` days, and the combined lead-time std dev (independent legs) is `σ_L_total = sqrt(3.0² + 1.0²)`. Recompute `SS_centralized` with these numbers.
10. How much of the pooling benefit from Part C survives once you account for the extra transfer leg? Write a two- or three-sentence recommendation to the VP: is centralizing still worth it here, worth it only with a faster/more reliable transfer leg, or not worth it at all for this SKU?

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|---------------|
| Mechanics | Decentralized and centralized SS computed correctly | Same, plus the naive square-root-law comparison computed explicitly, not skipped |
| Diagnosis | Notices the naive law is "off" | Correctly identifies *which term* breaks the naive law's assumption and explains why in terms of how means vs. variances aggregate |
| Dollar framing | Reports unit differences only | Converts every comparison to annual holding-cost dollars, and states the naive law's error in dollars/units of under-protection |
| Network thinking | Ignores the transfer-leg trade-off | Engages with the stretch and gives a specific, numbers-backed recommendation, not just "it depends" |

## Submission

Commit `challenge-02.md` (write-up) and `challenge-02.sql` / `challenge-02.py` (your calculations) to your portfolio under `c40-week-05/challenge-02/`.
