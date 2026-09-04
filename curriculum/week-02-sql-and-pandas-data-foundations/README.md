# Week 2 — SQL & pandas Data Foundations

> **Goal:** by Sunday you have a real, normalized PostgreSQL database modeling Crunch Gear's multi-echelon network — suppliers, plants, DCs, lanes, SKUs, orders, and shipments — seeded with real-looking data, and you can join across all of it, aggregate it into business answers, and move it into pandas and back without ever touching a spreadsheet.

Welcome back to **C40 · Crunch Chain**. Last week you reasoned about supply chains on paper: you mapped nodes and flows by hand, computed OTIF and cash-to-cash cycle time with a calculator, and sketched network designs on a blank page. That was the right way to *learn* the concepts — but no real operations team runs on paper. This week those same ideas — suppliers, lanes, orders, shipments, KPIs — become actual tables in an actual database, and the KPIs you computed by hand become queries you can rerun on tomorrow's data in milliseconds.

This is also the week the course's **data rule** stops being a sentence in the syllabus and starts being muscle memory: every record this course ever stores, manages, or queries lives in **SQL (PostgreSQL 16, SQLite fallback)**, and any modeling on top of it happens in **Python (pandas)**. Never a spreadsheet. By the end of this week you'll understand *why*, not just that it's the rule — the moment you try to answer "what's our OTIF by region, broken out by carrier, for orders that shipped from Newark" with a spreadsheet versus a `GROUP BY`, the rule stops feeling arbitrary.

We keep using **Crunch Gear**, the fictional outdoor-apparel company from Week 1 — same customers (TrailStop Outfitters, Ridgeline Retail, Prairie Supply Co., Summit & Sea), same regions, same product line. This week you build the real database behind that story: 2 contract-manufacturing plants, 4 distribution centers, 3 raw-material suppliers, 8 SKUs, a month of wholesale orders, and the shipments that fulfilled them — messy enough (short-shipments, late deliveries, split shipments) to make every query in this week's lectures a real business answer, not a toy example.

## Learning objectives

By the end of this week, you will be able to:

- **Design** a normalized PostgreSQL schema for a multi-echelon network — suppliers, sites (plants/DCs), lanes, SKUs, orders, and shipments — with primary keys, foreign keys, and constraints that make bad data impossible to insert.
- **Load and seed** that schema into PostgreSQL (and SQLite as a zero-setup fallback) from a real seed script, and verify the load with row-count and referential-integrity checks.
- **Query** the network fluently: multi-table joins across the full chain, `GROUP BY` aggregation for business rollups, and window functions for running inventory balances and lead-time percentiles.
- **Move data between SQL and pandas** — read query results with `pandas.read_sql`, do further modeling in a DataFrame, and write results back to Postgres with `to_sql` — while keeping the database, never a spreadsheet, as the system of record.
- **Compute operational KPIs** — fill rate, OTIF, and lead-time percentiles — directly in SQL, then independently recompute them in pandas and reconcile the two, catching the exact kind of silent bug (a join fan-out, a duplicate row) that makes cross-checking your tools a required habit, not a nice-to-have.

## Standards this week meets

| Bar | What this week is measured against |
| --- | --- |
| University | `ISM 4400` — work with the operating records of a supply chain — orders, shipments, receipts, inventory — and produce a performance report from them. |
| Industry | Take a folder of raw system exports, load it into a schema that refuses bad data, and hand back the KPI pack by Friday without a spreadsheet in the chain. |
| Beyond the bar | The same KPI is computed twice, independently in SQL and in pandas, over a discrepancy planted on purpose that the learner has to find and explain rather than paper over — `challenges/challenge-02-reconcile-sql-and-pandas-kpis.md` |

## Prerequisites

- Week 1 complete — you should recognize every entity in this week's schema (supplier, plant, DC, lane, order, shipment) and every KPI (OTIF, fill rate) from Lecture 2's hand computations.
- **PostgreSQL 16+** installed and runnable (`psql --version`), or **SQLite 3.35+** as a fallback (`sqlite3 --version`). See [`resources.md`](./resources.md) if you haven't installed either yet.
- **Python 3.10+ with pandas** (`pip install pandas`), plus `psycopg2-binary` or `sqlalchemy` if you're using Postgres from Python.
- Basic SQL (`SELECT`, `WHERE`, `JOIN`) helps but isn't required — Lecture 1 starts from `CREATE TABLE` and Lecture 2 builds up joins and aggregation from first principles. If SQL syntax is genuinely new to you, [C33 Crunch SQL](../../../C33-CRUNCH-SQL/) Weeks 1–2 are the ideal companion.

## Setup

1. Create a database (Postgres) or a file (SQLite) named `crunch_chain` — full commands are in [Exercise 1](./exercises/exercise-01-seed-the-network-database.md), which also contains the complete schema and seed data you'll run first thing.
2. Everything else this week — lectures, exercises, challenges, mini-project — queries that one seeded database. Seed it once, on Monday, and you're set up for the whole week.

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it — in particular, **run Exercise 1's seed script before reading Lecture 2**, since Lecture 2's queries are written directly against that data.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-modeling-the-chain-in-sql.md](./lecture-notes/01-modeling-the-chain-in-sql.md) | Designing the schema: suppliers, sites, lanes, SKUs, orders, shipments — keys, constraints, and why the database is the system of record | 2.5h |
| 2 | [lecture-notes/02-operations-analytics-in-sql.md](./lecture-notes/02-operations-analytics-in-sql.md) | Joins across the chain, `GROUP BY` aggregation, and window functions for running inventory and lead-time percentiles | 2.5h |
| 3 | [lecture-notes/03-sql-to-pandas-workflow.md](./lecture-notes/03-sql-to-pandas-workflow.md) | Reading query results into pandas, writing back to Postgres, and why this replaces the spreadsheet workflow entirely | 2h |
| 4 | [exercises/exercise-01-seed-the-network-database.md](./exercises/exercise-01-seed-the-network-database.md) | Run the full schema + seed script; verify counts and referential integrity | 1.5h |
| 5 | [exercises/exercise-02-join-orders-to-shipments.md](./exercises/exercise-02-join-orders-to-shipments.md) | Join orders through order lines, shipments, and shipment lines into a per-order fulfillment view | 1.5h |
| 6 | [exercises/exercise-03-rolling-inventory-with-windows.md](./exercises/exercise-03-rolling-inventory-with-windows.md) | Running inventory balances and lead-time percentiles with window functions | 1.5h |
| 7 | [challenges/challenge-01-build-an-otif-report-query.md](./challenges/challenge-01-build-an-otif-report-query.md) | Build a single query pack that reports OTIF, fill rate, and perfect order rate, sliced by region and carrier | 1.5h |
| 8 | [challenges/challenge-02-reconcile-sql-and-pandas-kpis.md](./challenges/challenge-02-reconcile-sql-and-pandas-kpis.md) | Compute the same KPIs in SQL and pandas independently — find and fix the discrepancy | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Model and load a multi-echelon network into Postgres from raw files, then produce a KPI snapshot query pack | 4h |
| 10 | [homework.md](./homework.md) | Extra practice, spaced across the week | 4.5h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official/free references + tools to install | — |

## Weekly schedule

Adds up to roughly the course's **~28 hr/week full-time pace**. Adjust to your own pace per the syllabus.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|-------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Schema design; seed the database | 2.5h | 1.5h | 0h | 0.5h | 1h | 0h | 5.5h |
| Tuesday | Joins across the chain | 1h | 1.5h | 0h | 0.5h | 1h | 0h | 4h |
| Wednesday | Aggregation & window functions | 1.5h | 1.5h | 0h | 0.5h | 1h | 1h | 5.5h |
| Thursday | SQL-to-pandas workflow | 2h | 0h | 1.5h | 0.5h | 1h | 1h | 6h |
| Friday | OTIF report + SQL/pandas reconciliation | 0h | 0h | 1.5h | 0.5h | 1h | 1h | 4h |
| Saturday | Mini-project | 0h | 0h | 0h | 0h | 0.5h | 1h | 1.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **7h** | **4.5h** | **3h** | **3.5h** | **5.5h** | **4h** | **27.5h** |

## By the end of this week you can…

- Write a `CREATE TABLE` schema for a multi-echelon network with correct keys and constraints, from a plain-English description of the business.
- Join six or more tables together to answer a real fulfillment question without double-counting rows from a fan-out.
- Compute a running inventory balance and a lead-time percentile using window functions, and explain why a window function is the right tool instead of a self-join or a spreadsheet running-total formula.
- Move a query result into pandas, transform it, and write it back to Postgres — and explain, concretely, why this loop never needs Excel.

## Up next

[Week 3 — Demand forecasting fundamentals](../week-03-demand-forecasting-fundamentals/) — now that the network's data lives in real tables, you'll pull order history out of it to forecast what customers order next.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
