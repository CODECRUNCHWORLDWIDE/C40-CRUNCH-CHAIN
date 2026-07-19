# Week 10 — Resources

Curated references and tools. You don't need to read everything here — the lectures are self-contained — but these are worth bookmarking for after the course, and for when you want a second explanation of a concept.

## Install / setup

- **PostgreSQL 16+** — primary engine this week (window functions, used in every SQL task, are identical across engines, but Postgres is the course default). Install guide: <https://www.postgresql.org/download/>
- **SQLite 3.35+** — zero-setup fallback; window function syntax used this week is fully supported from 3.25 onward. Install guide: <https://www.sqlite.org/download.html>
- **Python 3.10+** with `pandas` — required for Exercise 2 and both challenges (the chase/level/mixed and scenario simulations are iterative, natural in Python, awkward in pure SQL):
  ```bash
  pip install pandas
  ```
- A text editor and terminal — nothing else is required.

## S&OP and IBP — concepts and process

- **APICS/ASCM (Association for Supply Chain Management)** — the standard professional body for supply chain planning; their CPIM and CSCP body-of-knowledge materials cover S&OP and aggregate planning in depth: <https://www.ascm.org/>
- **Council of Supply Chain Management Professionals — glossary** — quick, authoritative definitions for every term this week uses (S&OP, IBP, safety stock, aggregate planning): <https://cscmp.org/CSCMP/Educate/SCM_Definitions_and_Glossary_of_Terms.aspx>
- **Gartner — Supply Chain research hub** — industry analysis and maturity models for S&OP/IBP adoption (some content requires a free account): <https://www.gartner.com/en/supply-chain>

## Aggregate planning — the underlying operations research

- **Investopedia — overtime, labor cost, and workforce planning basics** (general business background, not SCM-specific, but useful context for the hiring/layoff/overtime cost trade-offs this week's lectures build on): <https://www.investopedia.com/>
- Any standard **Operations Management** textbook (Heizer & Render, or Chopra & Meindl's *Supply Chain Management*) covers aggregate planning, chase/level/mixed strategies, and the classic linear-programming formulation of the problem in far more mathematical depth than this week goes into — worth a library visit if the topic clicks for you and you want the LP formulation directly (which connects to Week 9's optimization work).

## SQL — window functions (the technical core of this week)

- **PostgreSQL — Window Functions tutorial:** <https://www.postgresql.org/docs/current/tutorial-window.html>
- **PostgreSQL — Window Functions full reference (`SUM() OVER`, `PARTITION BY`, `ORDER BY`, frame clauses):** <https://www.postgresql.org/docs/current/functions-window.html>
- **SQLite — Window Functions:** <https://www.sqlite.org/windowfunctions.html>
- **PostgreSQL — `LEAST` / `GREATEST` and conditional expressions** (used throughout this week's reconciliation queries): <https://www.postgresql.org/docs/current/functions-conditional.html>

## Python / pandas — for the iterative simulations

- **pandas — Rolling/expanding/cumulative computations:** <https://pandas.pydata.org/docs/user_guide/window.html>
- **pandas — `DataFrame` construction from a list of dicts** (the pattern Lecture 2's worked example uses to build each month's simulation row): <https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.html>

## Where this week connects

- **Week 3–4 (this course)** — the statistical forecasting methods behind `stat_forecast_units` in `demand_plan`.
- **Week 5 (this course)** — safety stock and service level, the concept behind every `safety_stock_target` this week.
- **Week 9 (this course)** — LP/MIP optimization; the constrained plan you built by hand in Challenge 1 is exactly the kind of problem a solver (PuLP, SciPy) automates once the number of decision variables grows past what's comfortable to reason about manually.
- **[C41 Crunch Excel](../../../C41-CRUNCH-EXCEL/)** — if you're curious what an S&OP cycle looks like when it *is* run in spreadsheets (most companies' legacy process), that course covers the tool, not the discipline. This week is deliberately the opposite: the discipline, run correctly, in SQL and Python.
