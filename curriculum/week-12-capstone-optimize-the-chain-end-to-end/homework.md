# Week 12 — Homework

Five problems, ~4 hours total, spread across the week. These reinforce the lectures with variations on the capstone pipeline — a new constraint, a sensitivity question, an LP you formulate from a blank page, and two written-reasoning tasks. Commit each. All problems use the schema and pipeline from this week's lectures and exercises unless stated otherwise.

---

## Problem 1 — Ten warm-up questions on the network (40 min)

Answer each with one SQL query against the schema from Lecture 1 §3, plus a one-sentence answer. Put queries and answers in `warmups.sql` with a `-- N` comment above each.

1. Which DC has the lowest fixed cost, and what is it?
2. Which single DC→region lane is the most expensive per unit?
3. Which single DC→region lane is the cheapest per unit?
4. What is the combined monthly capacity of all three DCs?
5. Which plant has the lower unit production cost for `SKU-200` (Summit Down Parka), and by how much per unit?
6. What is the total inbound lead time difference (in days) between sourcing `SKU-100` from Hai Phong vs. Piedmont into Memphis DC?
7. If Crunch Gear had to serve the entire West region from Austin East instead of Reno DC, what would the extra per-unit cost be?
8. Which region has the highest per-unit freight cost from every single DC (i.e., is the most expensive region to serve, DC-independent)?
9. Sum the `demand_share` column in `skus` — what should it equal, and why does that matter for the demand generator?
10. Which two DC→region lanes are tied (or closest) in cost, and would that make the LP's solution non-unique in principle (multiple optimal plans with the same total cost)?

---

## Problem 2 — A new region (60 min)

Crunch Gear is entering a **fifth region: Mountain** (anchor customer: a new wholesale account in Denver, CO), with forecast demand of **5,200 units/month**, to be served from whichever existing DC(s) make sense — no new DC is being built.

1. Add plausible `dc_region_lanes` rows for `MTN` from all three existing DCs (`AUS`, `MEM`, `REN`) — use your own judgment for freight cost and lead time based on rough geography (Reno and Austin are both plausibly closer to Denver than Memphis; justify your numbers in one sentence each).
2. Re-run the network LP from Lecture 2 §4 with `MTN` added to `REGIONS` and `DEMAND`, keeping all other regions and demand at their existing values.
3. Report the new optimal plan and total outbound cost. Does adding a fifth region change any of the *other four* regions' assignments, or does Mountain simply slot into whichever DC has leftover capacity without disturbing the rest? Explain why, in terms of the LP's structure, one or the other is likely (or actually happened, if you ran it).

**Deliver** `new-region.py` with the updated LP and `new-region.md` with the findings.

---

## Problem 3 — Sensitivity: the price of the service-level target (50 min)

Everything this week assumed a 95% cycle service level (`Z = 1.645`). Re-run `compute_inventory_policy()` (Lecture 2 §3) at three service levels — **90%** (`Z ≈ 1.282`), **95%** (`Z ≈ 1.645`), and **99%** (`Z ≈ 2.326`) — using the **optimized** DC assignment from Exercise 3.

1. Report total network-wide safety-stock holding cost at all three levels.
2. Compute the cost of moving from 95% to 99% service, in dollars per month. Is the relationship between service level and safety-stock cost linear, or does cost accelerate as service level approaches 100%? Explain why, referencing how `Z` itself grows as the service-level target approaches 1.0 (look up a few more values on the standard normal table — `Z(99.9%) ≈ 3.09` — and notice how fast it's climbing).

**Deliver** `service-level-sensitivity.py` and `service-level-sensitivity.md` with the three-level comparison table and the written explanation.

---

## Problem 4 — Explain the LP to someone who's never seen one (40 min)

In `explain-the-lp.md`, write no more than 400 words explaining Lecture 2 §4's network-flow LP to a warehouse operations manager who has never written a line of code and doesn't know what "linear programming" means. You may **not** use the words "decision variable," "objective function," "constraint," or "solver" — find plain-language equivalents for all four concepts and use a concrete example from this week's network (the Memphis-capacity-forces-a-2,000-unit-reroute example from Lecture 1 §5 works well) to make it tangible.

This is harder than it sounds, and that's the point — Lecture 3 argued that translating technical work into something a non-technical stakeholder trusts *is* the job, not an afterthought to it.

---

## Problem 5 — A one-query cost audit (50 min)

Someone on finance asks: "if we just look at cost per unit shipped, ignoring fixed costs, which DC is currently our cheapest to ship out of, and which is our most expensive?" In `cost-audit.sql`, write a **single query** (CTEs allowed) against `dc_region_lanes`, weighted by the naive baseline volumes from Lecture 1 §5 (AUS 24,000, MEM 0, REN 6,500 — note MEM has zero volume in the current baseline, so it can't have a defined weighted-average cost; handle that edge case explicitly rather than letting it silently show as `NULL` or `0` with no explanation), that returns each DC's current volume-weighted average outbound cost per unit.

**Deliver** the query plus 2–3 sentences: does the DC with the *lowest* weighted average cost per unit correspond to the DC the LP chose to load up to capacity in Lecture 2 §4? If yes, is that a coincidence, or exactly what you'd expect an optimizer to do? If a DC has no volume in the baseline (and therefore no defined weighted-average cost), say what that emptiness itself already tells a finance stakeholder before the LP even runs.

---

## Time budget

| Problem | Time |
|--------:|----:|
| 1 | 40 min |
| 2 | 60 min |
| 3 | 50 min |
| 4 | 40 min |
| 5 | 50 min |
| **Total** | **~4 h** |

After homework, take the [quiz](./quiz.md) and ship the [mini-project](./mini-project/README.md).
