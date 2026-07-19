# Exercise 2 — Nearest-Neighbor Routing

**Goal:** Implement the nearest-neighbor heuristic from Lecture 2 yourself, first as an unconstrained single-vehicle route, then as a capacity-constrained multi-truck (CVRP) solution — and verify your numbers against the lecture's worked example.

**Estimated time:** 90 minutes.

## Setup

Confirm the seed is loaded and pull it into pandas:

```python
import math
import sqlite3          # or: import psycopg2 / sqlalchemy, if you're on Postgres
import pandas as pd

conn = sqlite3.connect("crunch_chain_wk7.db")
stops = pd.read_sql("SELECT * FROM delivery_stops ORDER BY stop_id", conn)
assert len(stops) == 13
print(stops)
```

Create `solutions.py` and write each task as a function or clearly separated block with a comment `# Task N`.

## Tasks

1. **Distance function.** Write `dist(a_id, b_id)` returning the Euclidean distance in miles between two `stop_id`s, using the `x_miles`/`y_miles` columns. *(Expected: `dist(0, 1)` returns exactly **17.0** — verify this before moving on; it's a Pythagorean triple, 8-15-17, and a wrong distance function will not hit it.)*

2. **Distance matrix (sanity check).** Build a 13×13 matrix (a dict of dicts, or a `numpy` array) of the distance between every pair of stops. Confirm it's symmetric: `dist(3, 7) == dist(7, 3)`.

3. **Unconstrained nearest-neighbor.** Implement `nearest_neighbor_route(stop_ids, start=0)` exactly as in Lecture 2, ignoring truck capacity entirely (pretend one infinite truck does the whole run). Run it on all 13 stops. *(Expected total distance: **243.8 miles**, ±0.1 for rounding. Expected route order: Austin East DC → Buda → Kyle → San Marcos → New Braunfels → Lockhart → Bastrop → Pflugerville → Round Rock → Georgetown → Cedar Park → San Antonio North → San Antonio South → Austin East DC.)*

4. **Capacitated nearest-neighbor.** Implement `capacitated_nearest_neighbor(stop_ids, demand, capacity, start=0)` as in Lecture 2, with `capacity = 120`. Print each route with its total load. *(Expected: **4 routes**, total distance **345.7 miles**, ±0.5. No route's load should exceed 120.)*

5. **Utilization check.** For each of your 4 capacitated routes, compute `load / 120` as a percentage — this is the truck's **capacity utilization**. Which route is best-utilized? Which is worst? *(A truck running at 70% utilization is carrying "empty air" the other 30% of its capacity — a real cost, even though it doesn't show up as a line item anywhere.)*

6. **What if capacity were 150 instead of 120?** Re-run Task 4 with `capacity = 150`. How many routes does it take now, and what's the new total distance? Does adding capacity always reduce total distance — why or why not?

## Expected result (spot checks)

- Task 1 → `dist(0, 1) == 17.0` exactly.
- Task 3 → 243.8 miles, single loop, all 12 stops.
- Task 4 → 4 routes, 345.7 miles total, every route's load ≤ 120.
- Task 6 → fewer than 4 routes possible at capacity 150 (total demand 409 cases; `409 / 150 = 2.7`, so at least 3 routes).

## Done when…

- [ ] `solutions.py` runs top to bottom with no errors and prints all 6 tasks' results.
- [ ] Task 1's `dist(0, 1)` is exactly 17.0 (confirms your formula is right before you trust anything downstream).
- [ ] Task 3's total distance is within 0.1 miles of 243.8.
- [ ] Task 4's total distance is within 0.5 miles of 345.7, and every route respects the 120-case capacity.
- [ ] You can explain in one sentence why Task 4's total distance (345.7) is *higher* than Task 3's (243.8), even though Task 4 uses more vehicles.

## Stretch

- Plot the 13 stops (matplotlib, `x_miles`/`y_miles`) with the depot marked distinctly, and draw your 4 capacitated routes in different colors. Does the routing look sensible by eye, or can you spot an obviously bad zigzag?
- Try starting the unconstrained nearest-neighbor from a *different* stop (not the depot) and see how much the total distance changes. What does that tell you about how sensitive nearest-neighbor is to its starting point?

## Submission

Commit `solutions.py` to your portfolio under `c40-week-07/exercise-02/`.
