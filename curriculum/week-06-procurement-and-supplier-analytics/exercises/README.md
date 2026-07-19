# Week 6 Exercises

Three exercises, meant to be done in order, all against the `purchase_orders` seed table from this week's [README](../README.md). Each builds directly on a lecture:

| Exercise | Builds on | Focus |
|---|---|---|
| [01 — Build a Spend Cube in SQL](./exercise-01-build-a-spend-cube-in-sql.md) | Lecture 1 | `GROUP BY`, `GROUPING SETS`, Pareto/cumulative percentage |
| [02 — Build a Supplier Scorecard](./exercise-02-build-a-supplier-scorecard.md) | Lecture 2 | Weighted scoring, min-max normalization, SQL + pandas |
| [03 — Price-Variance Analysis](./exercise-03-price-variance-analysis.md) | Lecture 3 (and Lecture 1's cost thinking) | Actual vs. standard cost, favorable/unfavorable variance |

## How to work these

1. Make sure the `purchase_orders` seed from the week [README](../README.md) is loaded before starting.
2. Write real SQL and/or Python — every exercise expects working code, not a description of what you'd do.
3. Each exercise states its **expected output** — check your numbers against it before moving on. If yours don't match, the bug is almost always a `GROUP BY` that's grouping the wrong grain (per-order vs. per-unit) — reread the "watch for" note in the exercise.
4. Keep your solutions — Exercise 2's scorecard function and Exercise 3's variance query both get reused (on different suppliers) in this week's challenges and mini-project.

No solutions are published in this folder — check your own results against the expected output shown in each file, per this course's honor-system policy.
