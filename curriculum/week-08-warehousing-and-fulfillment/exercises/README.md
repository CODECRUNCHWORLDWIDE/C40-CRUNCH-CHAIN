# Week 8 — Exercises

Three exercises, done in order. Exercise 1 classifies the catalog by velocity, Exercise 2 turns that classification into a slotting plan and a travel-distance number, Exercise 3 layers batching on top and measures the trip reduction. Each one reuses the previous exercise's output — do them in sequence.

## How to work these

- Use the `warehouse_skus`, `pick_lines`, and `warehouse_slots` tables seeded in the [week README](../README.md). Confirm the counts first: `20` SKUs, `225` pick lines, `24` slots.
- Do the SQL version and the pandas version where both are asked for — they should agree to rounding. A mismatch almost always means an arithmetic slip (usually the `× 2` round-trip factor, or forgetting the `ROUND()`) in one of them; use the disagreement to find your bug.
- Keep a running script or notebook per exercise (`exercise-01.sql` / `exercise-01.py`, etc.) — you will reuse this exact logic, barely modified, in the mini-project.
- Show your work for anything that isn't a single query — a one-line comment above a number ("used the Lecture 2 re-slot distances") saves a reviewer from re-deriving your reasoning.

## Files

| Exercise | Focus | Time |
|---|---|---|
| [exercise-01-abc-analysis-for-slotting.md](./exercise-01-abc-analysis-for-slotting.md) | Classify all 20 SKUs into A/B/C tiers from the pick-line profile | 1.5h |
| [exercise-02-pick-path-distance.md](./exercise-02-pick-path-distance.md) | Compute current vs. re-slotted total pick-travel distance | 1.5h |
| [exercise-03-order-batching-heuristic.md](./exercise-03-order-batching-heuristic.md) | Batch a day's orders into waves and measure the trip reduction | 1h |

## Done when…

- [ ] You have an A/B/C class for all 20 SKUs, computed from a real cumulative-percentage column, not eyeballed.
- [ ] You have a total pick-travel-distance number for Austin East DC's current slotting **and** for a velocity-based re-slot, and can state the percentage reduction.
- [ ] You've batched at least one real day of orders into a single wave and can state how many fewer trips it took versus discrete picking.
