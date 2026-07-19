# Week 2 — Challenges

Two open-ended problems, done after the three exercises. Both build on the exact dataset from Exercise 1 — Challenge 1 asks you to design and build a full report query from a loose spec, and Challenge 2 puts you through a deliberately planted SQL-vs-pandas discrepancy you have to find and explain, not just fix.

1. **[Challenge 1 — Build an OTIF report query](challenge-01-build-an-otif-report-query.md)** — go beyond Lecture 2's single OTIF number: build one query pack that reports OTIF, fill rate, and perfect order rate, sliced by region and by carrier. *(~90 min.)*
2. **[Challenge 2 — Reconcile SQL and pandas KPIs](challenge-02-reconcile-sql-and-pandas-kpis.md)** — compute the same KPI two independent ways, in SQL and in pandas, and find why they disagree by design. *(~90 min.)*

## How these are judged

There's no single-number answer key for either challenge — the queries and code you write are yours to design, within the spec given. You're being judged on:

- **Correctness of the numbers**, verified against the dataset — a query that runs but returns a subtly wrong number (usually from a fan-out, a missing `COALESCE`, or an off-by-one date comparison) is a failed submission even if it "looks fine."
- **Structure of the SQL/pandas** — readable CTEs over five-deep nested subqueries; a pre-aggregated join over a fan-out you didn't notice.
- **Written reasoning**, where asked — Challenge 2 in particular is graded more on whether you correctly diagnose *why* two numbers disagree than on which number you ultimately trust.

Keep your work in `challenge-01.sql` / `challenge-02.py` (or `.sql` + `.md`, your choice) with results and reasoning written out, not just code. The reasoning is the point.
