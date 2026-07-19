# Week 3 — Exercises

Three guided exercises, ~45–90 min each. **Type every query and every line of pandas yourself** — this week's whole point is that you build every forecasting method from first principles, once, so you know exactly what's inside it before Week 4 hands the same job to a library.

1. **[Exercise 1 — Pull demand from SQL to pandas](exercise-01-pull-demand-from-sql-to-pandas.md)** — query `demand_history`, load it into a `DataFrame`, reshape it for time-series work.
2. **[Exercise 2 — Moving-average forecast](exercise-02-moving-average-forecast.md)** — build naive, seasonal naive, and moving-average forecasts in pandas.
3. **[Exercise 3 — Score forecast error vs. a baseline](exercise-03-score-forecast-error.md)** — compute MAE/MAPE/RMSE/bias for every method and rank them per SKU.

## Before you start

- You've completed all three lectures.
- You ran the seed from the [week README](../README.md) and `SELECT COUNT(*) FROM demand_history;` returns **624**.
- You have Python 3.10+ with `pandas` installed, and a way to connect to your database from Python: `psycopg[binary]` for Postgres, or the built-in `sqlite3` module for SQLite.
- You have a shell open: `psql crunchchain` (Postgres) or `sqlite3 crunchchain.db` (SQLite), plus a Python REPL or script file.

## Suggested workflow

- Open the exercise file beside your editor.
- For each task, write the SQL or pandas yourself before checking the "Expected" note — don't peek first.
- If a number surprises you, stop and figure out why before moving on. Every "surprise" this week (a lagging moving average, a zero-division in MAPE, a method that loses to something simpler) is a deliberate lesson, not a mistake in the data.
- Save your work as you go: `pull_demand.py`, `forecasts.py`, `score_errors.py`. You'll extend these directly in the challenges and mini-project, so keep them clean and reusable.

## Data rule reminder

All demand history lives in SQL. You pull it into pandas to compute — you never re-key it into a spreadsheet, and you never hand-copy numbers between tools. If a step feels like "I could just do this faster in Excel," that feeling is the thing this course is training you out of — write the two extra lines of pandas instead, because pandas scales to 50 SKUs (the mini-project) and 5,000 SKUs (a real job) the exact same way, and a spreadsheet does not.
