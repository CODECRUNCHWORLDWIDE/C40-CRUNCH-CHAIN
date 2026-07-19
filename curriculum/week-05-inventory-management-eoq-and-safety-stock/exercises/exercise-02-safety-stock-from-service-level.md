# Exercise 2 — Safety Stock from a Service Level

**Goal:** Turn a target cycle-service level into a z-score, a z-score into safety stock, and safety stock into a reorder point — for the full catalog, at three different service levels, so you can see exactly how expensive "more service" gets.

**Estimated time:** 1.5 hours.

## Setup

Same `skus` table. You'll need `scipy.stats.norm` for the pandas half (`pip install scipy` if you don't have it).

## Tasks

### Part A — by hand, one SKU

1. For **SKU 4 (Merino Base Layer Top)**, compute:
   - Mean daily demand `d̄ = annual_demand / 365`.
   - `σ_DLT = sqrt(lead_time_days * demand_std_daily² + d̄² * lead_time_std_days²)`.
   - Safety stock at a **95%** cycle-service level (`z = 1.645`).
   - The reorder point `ROP = d̄*lead_time_days + SS`.

   Show every intermediate number — don't skip to the answer. *(Expected: `d̄ ≈ 26.3`, `σ_DLT ≈ 60.0–61.0`, `SS ≈ 99–100`, `ROP ≈ 362–363`. If you're more than a couple of units off, recheck which term inside the square root you squared.)*

### Part B — SQL, whole catalog

2. Write one query that computes `daily_demand`, `sigma_dlt`, `safety_stock`, and `reorder_point` for **all 10 SKUs** at a 95% service level (`z = 1.645`), matching the structure from Lecture 2 section 7.

3. Duplicate the query (or parameterize it) for **90%** (`z = 1.28`) and **99%** (`z = 2.33`). You should end up with three safety-stock numbers per SKU.

### Part C — pandas, three service levels at once

4. Load `skus` into a DataFrame. Compute `sigma_dlt` once, then loop over `csl in [0.90, 0.95, 0.99]`, using `scipy.stats.norm.ppf(csl)` for each `z`, and add `ss_90`, `ss_95`, `ss_99` columns (and matching `rop_*` columns).

5. **Cost of the buffer.** For each SKU, compute `ss_holding_cost_95 = ss_95 * unit_cost * holding_pct` — the annual dollar cost of carrying that SKU's 95%-target safety stock. Sum it across the catalog. Do the same for `ss_holding_cost_99`. Report the **total dollar jump** across the whole catalog from moving every SKU from a 95% to a 99% target.

6. **Which SKU is most sensitive?** Compute, per SKU, the percentage increase in safety stock going from 95% to 99% (`ss_99/ss_95 - 1`). This percentage should be the *same for every SKU* — explain in one sentence why (hint: look at what `z_99/z_95` equals, and where `z` sits in the safety-stock formula).

## Expected results (spot checks)

- SKU 8 (Insulated Water Bottle) — from the lecture's worked example: `σ_DLT ≈ 107.6`, `SS_95 ≈ 177`, `ROP_95 ≈ 670`; `SS_99 ≈ 251`, `ROP_99 ≈ 744`.
- The ratio `SS_99 / SS_95` should be **identical across all 10 SKUs** (≈ `2.33/1.645 ≈ 1.42`) — a 42% increase in safety stock for every SKU, regardless of that SKU's own demand or lead-time variability.

## Done when…

- [ ] Part A's by-hand SKU 4 numbers are within rounding of the expected range.
- [ ] The SQL query in Part B runs for all three service levels and matches the pandas output in Part C.
- [ ] Task 6's ratio is confirmed to be constant across SKUs, with a one-sentence explanation of why.
- [ ] You can state, in dollars, what the 95%→99% service-level jump costs Crunch Gear across the whole catalog.

## Stretch

- SKU 7 (Crunch Trail Backpack) has the longest lead time in the catalog (21 days) and a comparatively high `lead_time_std_days` (4.0). Decompose its `σ_DLT` into the two terms under the square root — how much of its total variability comes from demand noise vs. lead-time noise? What would you recommend Crunch Gear's ops team focus on improving for this SKU: tightening the forecast, or pressuring the supplier for more reliable lead times?

## Submission

Commit `exercise-02.sql` and `exercise-02.py` to your portfolio under `c40-week-05/exercise-02/`.
