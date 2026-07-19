# Week 2 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific question comes up.

## Install (needed before Exercise 1)

- **PostgreSQL 16+** — this course's primary data engine: <https://www.postgresql.org/download/> · macOS: [Postgres.app](https://postgresapp.com/) is the easiest. Linux: `sudo apt install postgresql` / `sudo dnf install postgresql-server`. Windows: the EDB installer.
- **SQLite 3.35+** — the zero-setup fallback; ships on macOS and most Linux already: <https://www.sqlite.org/download.html>. Check with `sqlite3 --version`. Note: window functions and `PERCENTILE_CONT` need a reasonably modern SQLite (3.25+ for window functions, 3.38+ for some ordered-set aggregates) — if `PERCENTILE_CONT` isn't available in your SQLite build, do this week's percentile queries in Postgres instead.
- **Python 3.10+ with pandas** — `pip install pandas`. Add `psycopg2-binary` (Postgres driver) or `sqlalchemy` (nicer `read_sql`/`to_sql` interface) — `pip install psycopg2-binary sqlalchemy`.
- **A GUI (optional)** — [DBeaver](https://dbeaver.io/) (free, both engines) or [pgAdmin](https://www.pgadmin.org/) (Postgres) — handy for browsing table structure and running quick checks alongside the terminal.
- If SQL syntax itself (not this week's schema, just basic `SELECT`/`WHERE`/`JOIN`) feels unfamiliar, [C33 Crunch SQL](../../../C33-CRUNCH-SQL/) Weeks 1–3 are the ideal companion — same two engines, same fundamentals, more time spent on syntax than this course has room for.

## Required reading (this week's core)

- **PostgreSQL — "Constraints":** <https://www.postgresql.org/docs/current/ddl-constraints.html>
  *Why: covers `PRIMARY KEY`, `FOREIGN KEY`, `CHECK`, and `UNIQUE` — every constraint type Lecture 1's schema uses, straight from the source.*
- **PostgreSQL — "Joins Between Tables":** <https://www.postgresql.org/docs/current/tutorial-join.html>
  *Why: the official tutorial on inner vs. outer joins — the exact distinction Lecture 2 Section 1 builds the fan-out warning on.*
- **PostgreSQL — "Window Functions" (tutorial chapter):** <https://www.postgresql.org/docs/current/tutorial-window.html>
  *Why: the official introduction to `OVER (...)`, written by the people who built it — read this alongside Lecture 2 Sections 3–5.*
- **PostgreSQL — "Window Functions" (full reference, incl. `PERCENTILE_CONT`):** <https://www.postgresql.org/docs/current/functions-window.html>
  *Why: the complete list of window functions beyond what this week covers (`FIRST_VALUE`, `NTH_VALUE`, and more) — useful once `SUM() OVER`, `RANK()`, `NTILE()`, and `LAG()` feel comfortable.*
- **pandas — "Merge, join, concatenate and compare":** <https://pandas.pydata.org/docs/user_guide/merging.html>
  *Why: the official guide to `pd.merge()` — read this before Challenge 2; the merge-cardinality trap the challenge plants is documented behavior, not a pandas bug.*

## Reference (keep in tabs)

- **pandas — `read_sql` API reference:** <https://pandas.pydata.org/docs/reference/api/pandas.read_sql.html>
- **pandas — `to_sql` API reference:** <https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.to_sql.html>
- **PostgreSQL — Aggregate Functions:** <https://www.postgresql.org/docs/current/functions-aggregate.html>
  *Why: `SUM`, `AVG`, `COUNT`, and the ordered-set aggregates (`PERCENTILE_CONT`, `PERCENTILE_DISC`) all live here.*
- **PostgreSQL — `COPY`:** <https://www.postgresql.org/docs/current/sql-copy.html>
  *Why: the fast, native way to load a CSV file straight into a table — useful for the mini-project's raw-file load if you'd rather not write individual `INSERT`s.*
- **SQLAlchemy — Engine configuration:** <https://docs.sqlalchemy.org/en/20/core/engines.html>
  *Why: the connection-string reference for `create_engine(...)` if Postgres via SQLAlchemy is new to you.*
- **Postgres vs. SQLite — feature differences that matter this week:** <https://www.sqlite.org/lang_corefunc.html> (SQLite's function reference — compare against Postgres's to spot what's missing, like `SERIAL`, which SQLite replaces with `INTEGER PRIMARY KEY AUTOINCREMENT`).

## Practice beyond this week's dataset

- **PostgreSQL Exercises — Joins and Aggregates sections:** <https://pgexercises.com/questions/joins/> and <https://pgexercises.com/questions/aggregates/>
  *Why: free, browser-based, graded drills on exactly this week's two hardest skills.*
- **Mode Analytics SQL Tutorial — Window Functions:** <https://mode.com/sql-tutorial/sql-window-functions/>
  *Why: a second, differently-worded explanation of window functions — useful if Lecture 2's explanation didn't fully click on the first pass; sometimes a second phrasing is what makes it click.*
- **Kaggle — "Supply Chain" dataset search:** <https://www.kaggle.com/search?q=supply+chain+in%3Adatasets>
  *Why: real (if messier) public datasets — good for practicing this week's join/window patterns on data that isn't already cleaned for you.*

## Deeper background (optional this week)

- **Date, C.J., *An Introduction to Database Systems*** — the classic textbook on relational theory (normalization, keys, constraints) behind everything Lecture 1 does informally; most university libraries carry an edition.
- **"A Relational Model of Data for Large Shared Data Banks," E.F. Codd (1970), *Communications of the ACM*:** <https://dl.acm.org/doi/10.1145/362384.362685>
  *Why: the original paper that proposed the relational model this entire week's schema is built on — short, historically foundational, and still readable.*

## Glossary

| Term | Definition |
|------|------------|
| **System of record** | The one place the current, true answer to a business question lives; everything else is a view of it. |
| **`SERIAL`** | Postgres shorthand for an auto-incrementing integer primary key. |
| **Foreign key** | A column constrained to only hold values that exist as a primary key in another table. |
| **`CHECK` constraint** | A rule Postgres enforces on every row of a column (or combination of columns) at insert/update time. |
| **XOR constraint pattern** | A `CHECK` enforcing that exactly one of two (or more) nullable columns is filled in — never both, never neither. |
| **Header/line pattern** | Modeling a parent record (e.g., an order) and its repeatable child rows (e.g., order lines) as two linked tables. |
| **Ledger pattern** | Storing every event as an appended row (never overwritten) so any historical state can be recomputed by summing up to a point in time. |
| **Join fan-out** | A join that legitimately multiplies rows on the "many" side of a one-to-many relationship, silently inflating any aggregate computed after it if not handled carefully. |
| **`GROUP BY`** | Collapses multiple rows into one row per distinct group, for use with aggregate functions. |
| **Window function** | Computes a value per row using a "window" of related rows, without collapsing the original rows the way `GROUP BY` does. |
| **`PARTITION BY`** | Inside a window function, resets its calculation independently per group — the window-function analog of `GROUP BY`. |
| **`PERCENTILE_CONT`** | An ordered-set aggregate returning the interpolated value at a given percentile (e.g., 0.9 for P90). |
| **`RANK()` / `DENSE_RANK()`** | Window functions assigning an order-based rank to each row; `RANK()` leaves gaps after ties, `DENSE_RANK()` doesn't. |
| **`LAG()` / `LEAD()`** | Window functions that pull a value from the previous/next row within a partition's order. |
| **`pd.read_sql`** | Runs a SQL query and returns the result as a pandas DataFrame. |
| **`df.to_sql`** | Writes a DataFrame's rows into a SQL table; `if_exists` controls append/replace/fail behavior. |
| **Long/tidy data** | A table shape with one row per (entity, metric, period) combination, rather than one column per metric — the shape both SQL and pandas work with most naturally. |

---

*Broken link? Open an issue or PR.*
