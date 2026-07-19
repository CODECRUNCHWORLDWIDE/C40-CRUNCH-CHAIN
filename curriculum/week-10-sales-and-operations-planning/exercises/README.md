# Week 10 — Exercises

Three guided exercises. Do them in order — Exercise 2 assumes you can already read a supply-demand balance table (Exercise 1), and Exercise 3's gap analysis reuses both the balance table and the strategy comparison to recommend fixes.

1. **[Exercise 1 — Build a Supply-Demand Balance Table](exercise-01-build-a-supply-demand-table.md)** — SQL: join demand, supply, and inventory into a running monthly balance using a window function. *(~1.5h)*
2. **[Exercise 2 — Compare Aggregate-Plan Strategies](exercise-02-compare-aggregate-plans.md)** — Python: cost out chase, level, and mixed strategies for Trail Footwear's real numbers. *(~1.5h)*
3. **[Exercise 3 — Gap Analysis & Actions](exercise-03-gap-analysis-and-actions.md)** — SQL + written reasoning: find every breach and recommend a costed fix. *(~1h)*

## Before you start

- You've read all three lectures.
- You ran all four seed tables from the [week README](../README.md): `demand_plan` (18 rows), `supply_plan` (18 rows), `product_reference` (3 rows), `financial_targets` (18 rows).
- You have a shell open (`psql crunch_chain_wk10` or `sqlite3 crunch_chain_wk10.db`) **and** a Python environment with `pandas` installed — Exercise 2's strategy simulation is iterative and is Python-only, the same reasoning Lecture 2 walked through.

## Suggested workflow

- Write your query or function **before** running it, then compare the output to the "Expected" note in each exercise.
- Save SQL answers in a `solutions.sql` file, one query per task under a `-- Task N` comment; save Python answers in `solutions.py`.
- If a running-total window function gives you a number that looks wrong, check your `PARTITION BY` before anything else — the single most common bug this week is a running total that silently sums across *all three* product families instead of resetting per family.

## Note on the two engines

Every SQL task in Exercises 1 and 3 uses `SUM() OVER (PARTITION BY ... ORDER BY ...)`, standard window-function syntax supported identically on PostgreSQL 16 and SQLite 3.35+. No engine-specific rewriting needed this week.
