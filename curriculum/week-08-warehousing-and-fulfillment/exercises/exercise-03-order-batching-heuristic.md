# Exercise 3 — Order-Batching Heuristic

**Goal:** Group a real day's orders into a pick wave, measure how many fewer trips batching takes versus discrete (one-order-at-a-time) picking, and build a simple greedy heuristic that batches an arbitrary set of orders by SKU overlap.

**Estimated time:** 1 hour.

## Setup

Reuse your Exercise 2 re-slot (SKU → distance mapping). If you skipped straight here, at minimum recompute the velocity-based slot assignment from Lecture 2 Section 7 before starting — this exercise's distance numbers depend on it.

## Tasks

### Part A — one day, both picking styles

1. Pick **June 4, 2025** (`order_date = '2025-06-04'`) from `pick_lines`. Report `n_orders`, `n_lines`, and `n_distinct_skus` for that day.

2. **Discrete picking cost**: sum `2 * distance_ft` (your re-slotted distances) across every one of that day's pick lines — one round trip per line.

3. **Batched (single-wave) picking cost**: sum `2 * distance_ft` across only the **distinct** SKUs touched that day — one round trip per unique location, regardless of how many orders/lines needed it.

4. Report the trip count and travel-distance reduction from batching, both as a raw number and a percentage.

### Part B — a whole month, one wave per day

5. Repeat Tasks 2–3 for **every** `order_date` in the table (20 days), and sum the discrete and batched travel totals across all of them.

6. Convert both monthly totals to minutes (250 ft/minute) and report the total minutes saved by batching, on top of whatever slotting plan you're using (your Exercise 2 re-slot).

### Part C — a greedy batching heuristic

7. Real DCs don't necessarily wave an entire day at once — waves are usually capped at some number of orders (a cart only holds so many totes). Write a function `build_waves(orders, wave_size)` that:
   - Takes a list of orders (each with its list of SKUs) and a maximum orders-per-wave.
   - Groups orders into waves of at most `wave_size`, **in the order they arrive** (don't reorder or optimize which orders go together yet — that's the challenge, not this exercise).
   - For each wave, computes `distinct_skus_in_wave` and the resulting batched travel distance.

8. Run `build_waves` on June 4's seven orders at **wave sizes of 2, 4, and 7** (7 = the whole day in one wave, matching Task 3). Report total batched travel distance at each wave size. Confirm travel distance goes **down** as wave size goes **up** (more orders per wave = more chances to share a stop) — and note by how much the marginal improvement shrinks between wave-size 4 and wave-size 7 versus between 2 and 4.

## Expected results (spot checks)

- June 4: 7 orders, 26 lines, **11 distinct SKUs**.
- June 4 discrete travel: **2,302 ft**. June 4 batched (1 wave) travel: **1,310 ft** — a **43%** reduction.
- Whole-month totals (20 days, 1 wave/day): discrete ≈ **77.1 minutes**, batched ≈ **54.4 minutes** — a reduction of about **22.8 minutes (29.6%)**.

## Done when…

- [ ] You've computed both picking styles for June 4 and for the full month, and your numbers match the spot checks (or you can explain a deliberate deviation, e.g. a different re-slot from Exercise 2's stretch goal).
- [ ] `build_waves` runs correctly at three wave sizes and shows travel distance decreasing as wave size increases.
- [ ] You can state, in one sentence, the diminishing-returns pattern you saw between wave sizes 2→4 and 4→7.

## Stretch

- `build_waves` currently groups orders **in arrival order**, which is arbitrary. Modify it (or write a second version) to instead sort orders by their **most-common SKU** before batching, so orders needing the same popular items land in the same wave more often. Re-run the wave-size-4 case from Task 8 with this smarter grouping and check whether `distinct_skus_in_wave` (and therefore batched travel) drops further than the arrival-order version. This is exactly the kind of *how you group orders* question Challenge 1 and the mini-project build on.

## Submission

Commit `exercise-03.py` (this one leans pandas/Python — the wave-building loop is naturally iterative) to your portfolio under `c40-week-08/exercise-03/`.
