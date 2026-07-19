# Week 8 — Homework

Five problems, ~5 hours total, spread across the week. These reinforce all three lectures with **fresh numbers** — a different, smaller facility than Austin East DC — so you can't pattern-match your way through without actually applying the formulas. Commit each.

---

## Problem 1 — ABC analysis for Reno DC's climbing-gear micro-catalog (45 min)

Crunch Gear's smaller Reno DC stocks a 6-SKU climbing accessories line. This month's pick-line counts:

| SKU | Pick line count |
|---|---:|
| Trail Bandana | 48 |
| Carabiner 2-Pack | 32 |
| Freeze-Dried Meal Pouch | 20 |
| Bear Canister | 8 |
| Camp Saw | 6 |
| Satellite Messenger | 2 |

By hand first, then check with a short script: compute each SKU's `pct_of_total` and `cumulative_pct`, and assign an A/B/C class using the standard 80/95 cutoffs. **Deliver** `problem-01.md` with the full table and a one-sentence note on where the A/B cutoff falls (which two SKUs it falls between).

*(Expect Class A to contain exactly 2 SKUs, cumulative just under 70% — a smaller A class than Austin East DC's, because this catalog's top SKU dominates even harder.)*

---

## Problem 2 — Slot a 5-SKU, 5-slot facility (60 min)

A tiny satellite pickup counter has exactly 5 storage slots and 5 SKUs. Slot distances (one-way, feet): **15, 30, 45, 60, 75**. This month's pick counts: **SKU1 = 40, SKU2 = 25, SKU3 = 15, SKU4 = 8, SKU5 = 2**.

The counter's *current* (bad) assignment puts the **slowest** mover in the **closest** slot and works outward from there (SKU5→15ft, SKU4→30ft, SKU3→45ft, SKU2→60ft, SKU1→75ft) — a real thing that happens when a small location's shelving gets assigned by whoever unboxed first, not by velocity.

1. Compute total monthly pick-travel distance (feet) for the current assignment. Remember the round-trip `× 2`.
2. Design the velocity-optimal assignment (Lecture 2, Section 7's pairing rule) and compute its total pick-travel distance.
3. Report the reduction, in feet and as a percentage.

**Deliver** `problem-02.md` with both totals and the reduction. *(Expect the current assignment to cost roughly double the optimal one.)*

---

## Problem 3 — Batch four orders across the Problem 2 facility (45 min)

Using your **velocity-optimal** slot assignment from Problem 2, these four orders arrived the same day:

| Order | SKUs on the order |
|---|---|
| 101 | SKU1, SKU2 |
| 102 | SKU1, SKU3 |
| 103 | SKU2, SKU4 |
| 104 | SKU1, SKU2, SKU5 |

1. Compute total pick-travel distance if each order is picked **discretely** (one trip per line, 9 lines total across the four orders).
2. Compute total pick-travel distance if all four orders are picked as **one batched wave** (one trip per distinct SKU touched that day).
3. Report the trip-count reduction (lines vs. distinct SKUs) and the distance reduction (feet and percentage).

**Deliver** `problem-03.md` with both totals. *(Expect discrete picking to need 9 trips and batched picking to need 5 — one per distinct SKU across all four orders.)*

---

## Problem 4 — Size a picking crew for Memphis DC's peak Tuesday (60 min)

Memphis DC forecasts **2,850 order lines** for a peak Tuesday tied to a wholesale replenishment cycle. Target pick rate: **140 lines/hour**. Shift: **7 hours**, with a 20-minute break (**6.667 productive hours**). Target utilization: **78%**.

1. Convert the pick rate to minutes/line.
2. Compute total required picker-minutes for the forecast.
3. Compute usable minutes per picker at the target utilization.
4. Compute pickers needed (round up).
5. If Memphis DC can only get **3** pickers that day, compute the shortfall in minutes and convert it to lines.

**Deliver** `problem-04.py` (or `.md` with hand work shown) with all five numbers.

---

## Problem 5 — Memo: is ABC-based slotting worth the disruption? (60 min)

Reno DC's site manager pushes back on re-slotting: *"Alphabetical storage is simple — anyone on the team can find anything without checking a screen. ABC analysis sounds like a solution in search of a problem for a facility this small."*

Write a **200–300 word memo** (`problem-05.md`) responding to this position. You must:

- Reference at least one concrete number from **this week's Austin East DC case** (the 44,466 → 19,284 foot re-slot, or the combined slotting+batching labor reduction from Lecture 3) as evidence.
- Either defend the manager's position for Reno DC specifically (six SKUs is a very different scale than twenty), rebut it, or propose a **middle ground** — e.g., a partial re-slot of only the highest-velocity SKUs, preserving most of the "anyone can find anything" simplicity while capturing most of the travel savings.
- Back your recommendation with your own Problem 1 and Problem 2 numbers, not just Austin East's.

There's no single correct verdict here — the grading standard is whether your argument is backed by numbers you actually computed, not general opinion.

---

## Submission

Commit all five `problem-0N.*` files to your portfolio under `c40-week-08/homework/`. Total time budget: about 5 hours across the week — don't do all five in one sitting; let Problems 3 and 4 sit until after you've reviewed Lectures 2 and 3 a second time.
