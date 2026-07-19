# Exercise 2 — Measure Pick-Path Distance

**Goal:** Compute exactly how many feet of pick travel Austin East DC's *current* slotting costs, design a velocity-based re-slot from Exercise 1's classification, and compute how many feet the re-slot saves. Then look at one multi-line order and compare a naive pick path to a distance-sorted one.

**Estimated time:** 1.5 hours.

## Setup

Confirm the seed table is loaded:

```sql
SELECT COUNT(*) FROM warehouse_slots;  -- must print 24
```

You'll need Exercise 1's `pick_line_count` per SKU — recompute it here or reuse your Exercise 1 script; either is fine, just don't hand-copy numbers.

## Tasks

### Part A — current-slotting travel distance

1. Join `pick_line_count` (per SKU, from Exercise 1) against `warehouse_slots.current_sku_id` to get each SKU's current `distance_ft`.

2. Compute `travel_ft_current = pick_line_count * 2 * distance_ft` per SKU (remember: `distance_ft` is one-way; a pick is a round trip). Sum it for the whole catalog. This is Lecture 2, Section 6's calculation — reproduce it yourself rather than copying the lecture's number.

3. Convert the total to minutes at **250 ft/minute** (this week's standard walking-and-searching pace) and to labor-hours.

### Part B — re-slot by velocity

4. Using Exercise 1's `abc_class` (or the raw `pick_line_count` ranking), design a re-slot: rank SKUs by `pick_line_count` descending, rank `warehouse_slots` by `distance_ft` ascending (exclude the four empty reserve slots, `R5`–`R8`), and pair them position-for-position — the assignment-problem pattern from Lecture 2, Section 7.

5. Recompute total travel distance under the re-slot (same formula as Task 2, new slot assignments). Report the new total in feet, minutes, and hours.

6. **State the result as one sentence a warehouse manager could act on**: "Re-slotting Austin East DC by pick velocity would cut monthly pick-travel distance by ___ feet (___%), saving approximately ___ labor-minutes per month." Fill in your own computed numbers — don't copy the lecture's.

### Part C — one order's pick path

7. Pull all lines for `order_id = 9` from `pick_lines`, joined to the **re-slotted** locations from Task 4. List the SKUs in the order they appear in the `pick_lines` table (the "naive" order) alongside each one's `distance_ft`.

8. On this week's simplified **single-line layout** (every slot sits somewhere along one line running out from the pack station, per Lecture 2 Section 5), a picker visiting several stops on one trip only ever needs to walk out to the **farthest** stop and back — any stop closer than that is passed on the way, in either direction. So: total path travel = `2 × MAX(distance_ft)` across the order's stops, **regardless of the order you visit them in**. Compute that number for order 9's stops.

9. Now sort order 9's stops by `distance_ft` ascending and list the visiting sequence a picker would actually walk (nearest first, working outward, then straight back). Confirm its total distance matches Task 8's `2 × MAX(distance_ft)` figure exactly.

10. Explain, in a sentence, why a real two-dimensional warehouse floor does **not** share this property — what would have to be true about the layout (aisles branching off a main corridor, say) for the *order* you visit stops in to start mattering, the way it did in Week 7's vehicle-routing lecture?

9. Compare: does re-ordering the visit sequence change total travel distance on this week's one-dimensional layout? Explain, in a sentence, why a one-dimensional (single-line) layout behaves differently from a real two-dimensional warehouse floor here — and what would have to be true about the layout for visit order to matter.

## Expected results (spot checks)

- Current-slotting total travel: **44,466 feet** (≈177.9 minutes, ≈3.0 labor-hours) for the month.
- Velocity re-slot total travel: **19,284 feet** (≈77.1 minutes, ≈1.3 labor-hours) — a **56.6%** reduction.
- `Hydration Bladder 2L` (SKU 1, 55 picks) should land in a **Golden**-zone slot under your re-slot; `4-Season Mountaineering Tent` (SKU 20, 1 pick) should land in **Reserve**.

## Done when…

- [ ] You have a current-slotting total travel number and a re-slotted total travel number, both reproduced from scratch (not copied from the lecture).
- [ ] Task 6's one-sentence summary is filled in with your own numbers.
- [ ] You've worked through order 9's pick path both ways and can explain Task 9's one-dimensional-layout observation.

## Stretch

- Real DCs never re-slot the *entire* warehouse in one pass — moving inventory has its own labor cost. Suppose Austin East DC can only afford to physically move **5 SKUs** this week. Using your velocity ranking and the current-slotting distances, which 5 moves would you make first to capture the *most* travel-distance savings per move? (Hint: compute each SKU's `travel_ft_current - travel_ft_if_moved_to_its_optimal_slot` and rank by that delta — the biggest deltas are your highest-priority moves.)

## Submission

Commit `exercise-02.sql` and `exercise-02.py` to your portfolio under `c40-week-08/exercise-02/`.
