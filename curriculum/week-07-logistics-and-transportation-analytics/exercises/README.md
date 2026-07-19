# Week 7 — Exercises

Three guided exercises. Do them in order — Exercise 3 assumes the `on_time` logic from Exercise 1's warm-up, and Exercise 2 stands alone but is easiest once Lecture 2's code is fresh in your head.

1. **[Exercise 1 — Compute Cost per Lane](exercise-01-cost-per-lane.md)** — SQL aggregation: cost per lane, cost per pound, cost per unit shipped. *(~1h)*
2. **[Exercise 2 — Nearest-Neighbor Routing](exercise-02-nearest-neighbor-routing.md)** — build and verify a nearest-neighbor route in Python, unconstrained and capacity-constrained. *(~1.5h)*
3. **[Exercise 3 — Carrier On-Time Analysis](exercise-03-carrier-on-time-analysis.md)** — score every carrier on reliability and cost in one SQL query. *(~1h)*

## Before you start

- You've read all three lectures.
- You ran both seed tables from the [week README](../README.md): `SELECT COUNT(*) FROM shipments;` returns **124**, `SELECT COUNT(*) FROM delivery_stops;` returns **13**.
- You have a shell open (`psql crunch_chain_wk7` or `sqlite3 crunch_chain_wk7.db`) **and** a Python environment with `pandas` and `numpy` installed, since Exercise 2 is Python-only (routing heuristics are iterative — awkward in pure SQL, natural in a loop).

## Suggested workflow

- Write your query or function **before** running it, then compare the output to the "Expected" note in each exercise.
- Save SQL answers in a `solutions.sql` file, one query per task under a `-- Task N` comment; save Python answers in a `solutions.py` file.
- If a number surprises you, stop and figure out why before moving on — in freight and routing work, "that number looks wrong" is usually either a real bug or a real insight, and you want to know which before you ship the answer to a stakeholder.

## Note on the two engines

Every SQL task in Exercises 1 and 3 runs unchanged on PostgreSQL and SQLite — the queries only use `GROUP BY`, `CASE`, and standard aggregates, none of which differ between the two engines this week.
