# Week 11 — Exercises

Three guided exercises. Do them in order — Exercise 2 assumes you're comfortable reading `daily_ops`, and Exercise 3 reuses the rolling-window pattern from Lecture 2 directly.

1. **[Exercise 1 — Build a Risk-Scoring Model](exercise-01-risk-scoring-model.md)** — score all 16 network risks, build the matrix, shortlist "must mitigate first." *(~1.5h)*
2. **[Exercise 2 — Simulate a Disruption](exercise-02-disruption-simulation.md)** — measure the Andes Stitch Works fire's cost and recovery time directly from `daily_ops`. *(~1.5h)*
3. **[Exercise 3 — Automate a Daily Ops Pipeline](exercise-03-automate-a-daily-ops-pipeline.md)** — build a rolling-baseline, anomaly-flagging pipeline in SQL + Python. *(~1h)*

## Before you start

- You've read all three lectures.
- You ran both seed tables from the [week README](../README.md): `SELECT COUNT(*) FROM network_risk_register;` returns **16**, `SELECT COUNT(*) FROM daily_ops;` returns **180**.
- You have a shell open (`psql crunch_chain_wk11` or `sqlite3 crunch_chain_wk11.db`) **and** a Python environment with `pandas` and `numpy` installed — Exercise 3 is SQL-and-Python together.

## Suggested workflow

- Write your query or function **before** running it, then compare the output to the "Expected" note in each exercise.
- Save SQL answers in a `solutions.sql` file, one query per task under a `-- Task N` comment; save Python answers in a `solutions.py` file.
- This week's numbers are meant to be checked exactly — the disruption is a fixed, deterministic dataset, not a random simulation you run yourself, so your query's output should match the spot checks precisely (rounding aside).

## Note on the two engines

Exercise 1's queries run unchanged on PostgreSQL and SQLite. Exercise 2's queries run unchanged on both engines too. Exercise 3's SQL portion uses `STDDEV`, which SQLite lacks natively — the exercise routes that specific calculation through pandas instead and flags exactly where.
