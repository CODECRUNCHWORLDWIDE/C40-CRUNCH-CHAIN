# Challenge 1 — Slotting Optimization Under a Replenishment Constraint

**Goal:** Lecture 2 and Exercise 2 optimized slotting on pick-travel distance alone. Real golden-zone slots are small — they hold roughly one case of stock — so a fast-moving SKU doesn't just get picked from its slot constantly, it also has to be **refilled** constantly. This challenge makes you re-slot under a labor constraint on how many refills the golden zone can absorb, and shows that "purely by velocity" and "cheapest overall" are not always the same plan.

**Estimated time:** 1.5 hours.

## The constraint

Each golden-zone slot holds **one case** of the SKU assigned to it (`units_per_case`, from `warehouse_skus`). Every time that case empties, a forklift operator has to bring a fresh case in from bulk reserve storage — a **replenishment trip**. Austin East DC's putaway crew can absorb **at most 8 replenishment trips into the golden zone per month** without pulling a forklift operator off receiving duty (Lecture 1's putaway step, competing for the same labor).

A SKU's monthly replenishment-trip count is:

```
replenishment_trips = CEIL(units_picked_this_month / units_per_case)
```

## Tasks

1. **Compute replenishment trips for all 20 SKUs.** Join `SUM(qty_picked)` per SKU (from `pick_lines`) against `units_per_case` (from `warehouse_skus`), and compute `CEIL(units_picked / units_per_case)` in SQL (`CEIL()` on Postgres; SQLite needs `-(-units_picked / units_per_case)` integer-division trick, or do this step in pandas with `np.ceil`).

2. **Check the unconstrained plan.** Take the pure-velocity golden-zone assignment from Exercise 2 (the 6 highest-pick-count SKUs). Sum their `replenishment_trips`. Confirm it **exceeds** the 8-trip cap — by how much?

3. **Find a feasible plan.** You need to swap enough high-replenishment SKUs out of the golden zone (replacing them with the next-highest-velocity SKUs waiting in line) to bring total golden-zone replenishment trips to **8 or fewer**. There is more than one combination of swaps that satisfies the constraint — find **at least two** different feasible combinations before picking one.

4. **Cost each feasible combination.** For each combination from Task 3, recompute total pick-travel distance for the *entire* re-slotted catalog (golden + middle, following the same "sort by velocity, assign to sorted distance" pattern as Lecture 2 — the demoted SKUs need new middle-zone slots too, and that reshuffles the whole middle-zone ranking). Report the total pick-travel cost of each combination.

5. **Pick the cheapest feasible plan** and justify it in 2–3 sentences: which SKU(s) did you keep in the golden zone despite the constraint pushing you to consider removing them, and why was removing a *different*, lower-velocity SKU cheaper overall even though it wasn't the "obvious" highest-replenishment offender?

## A trap worth naming

It's tempting to solve the constraint by removing whichever SKU has the **most** replenishment trips, one at a time, until you're under the cap. That greedy rule is not guaranteed to minimize total pick-travel cost — a SKU with a middling trip count but *very* high pick velocity can be far more expensive to demote than a different SKU with the same trip count but much lower velocity. Compute both the "remove highest trip-count first" plan and your own cost-optimized plan from Task 4, and compare their total pick-travel costs directly. If they're not the same plan, that gap **is** the point of this challenge.

## Expected results (spot checks)

- The unconstrained golden-zone plan (SKUs 1–6) requires **12** replenishment trips/month — 4 over the 8-trip cap.
- At least one feasible combination brings the golden zone to **exactly 8** trips/month while adding **under 3,000 feet** of monthly pick-travel versus the unconstrained plan.
- A "remove the two highest trip-count SKUs" greedy plan is also feasible (hits 8 trips) but costs **noticeably more** total pick-travel than the best combination you found in Task 4 — if your greedy and optimized numbers come out equal, re-check which SKUs you actually swapped.

## Why this matters

Every optimization in this course so far (EOQ, safety stock, routing) had one clean objective. Real operations decisions almost always have **two** — here, pick-travel cost and replenishment labor — and the cheapest plan on one axis is rarely the cheapest plan once you account for both. Spotting that trap, and being able to quantify it instead of hand-waving "well, there's a trade-off," is what separates an analyst who runs one query and stops from one who actually gets the recommendation right.

## Submission

Commit `challenge-01.sql` and/or `challenge-01.py`, plus a short `challenge-01-writeup.md` with your Task 5 justification, to your portfolio under `c40-week-08/challenge-01/`.
