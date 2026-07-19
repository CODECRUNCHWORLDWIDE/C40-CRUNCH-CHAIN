# Week 12 — Exercises

Three exercises, strictly sequential — each one's output is a table or file the next one reads. This mirrors the capstone pipeline itself: assemble the data, run forecast + policy, then optimize and report. Do not skip ahead; Exercise 3 will not run without Exercise 2's output, and Exercise 2 will not run without Exercise 1's schema.

1. **[Exercise 1 — Assemble the Capstone Data Model](exercise-01-assemble-the-data-model.md)** — build and verify the full SQL schema and 24-month demand history from Lecture 1. *(~1.5h)*
2. **[Exercise 2 — Run the Forecast & Inventory Policy](exercise-02-run-forecast-and-policy.md)** — forecast every SKU-region pair, backtest it, and compute EOQ/safety stock/reorder point for each. *(~2h)*
3. **[Exercise 3 — Optimize & Report](exercise-03-optimize-and-report.md)** — build and solve the network LP, then produce a baseline-vs-optimized cost report. *(~2h)*

## Before you start

- You've read all three lectures, especially Lecture 1's schema and demand generator and Lecture 2's three pipeline functions.
- Python environment with `pandas`, `numpy`, `scipy`, `sqlalchemy`, and `pulp` installed (`pip install pandas numpy scipy sqlalchemy pulp`).
- A running PostgreSQL 16+ database (`crunch_chain_capstone`) or SQLite fallback (`crunch_chain_capstone.db`).

## Suggested workflow

- Keep one running Python script or notebook per exercise (`exercise_01.py`, `exercise_02.py`, `exercise_03.py`) rather than a scratch REPL session — the mini-project assembles all three into one pipeline script, and you'll be glad to have working code to start from.
- Every exercise has an "Expected outcome" section with specific numbers. If yours don't match, the bug is almost always in the seed data or a join key, not in your logic — check the sanity-check row counts from Lecture 1 first.
- Commit after each exercise. The three build directly on each other, and a broken Exercise 1 schema silently breaks Exercises 2 and 3 in ways that are hard to diagnose after the fact.
