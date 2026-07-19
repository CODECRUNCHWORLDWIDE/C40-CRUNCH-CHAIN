# Week 9 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific question comes up.

## Install first

- **Python 3.10+** with **PuLP** and **SciPy** — required for everything this week:
  ```bash
  pip install pulp scipy pandas
  ```
  PuLP ships with the open-source **CBC** solver bundled in — no separate solver download needed for anything in this course.
- **PostgreSQL 16+** — the course's primary engine, used for the Week 9 network seed and the mini-project: <https://www.postgresql.org/download/> · macOS: [Postgres.app](https://postgresapp.com/). Linux: `sudo apt install postgresql` / `sudo dnf install postgresql-server`. Windows: the EDB installer.
- **SQLite 3.35+** — the zero-setup fallback, ships on macOS and most Linux already: <https://www.sqlite.org/download.html>. Check with `sqlite3 --version`.
- **(Optional) matplotlib**, if you want to plot a feasible region for Lecture 1's geometry section: `pip install matplotlib`.

Verify your solver stack before Lecture 1:

```python
import pulp
print(pulp.listSolvers(onlyAvailable=True))   # should include 'PULP_CBC_CMD'

from scipy.optimize import linprog
print(linprog([1], A_ub=[[1]], b_ub=[10]).success)   # should print True
```

## Required reading (this week's core)

- **PuLP documentation:** <https://coin-or.github.io/pulp/>
  *Why: the primary modeling library for every lecture, exercise, and challenge this week — read the "Optimisation Concepts" and "Basic PuLP objects" pages before Lecture 1.*
- **SciPy — `linprog` reference:** <https://docs.scipy.org/doc/scipy/reference/generated/scipy.optimize.linprog.html>
  *Why: Lecture 1's second solving path, and the tool you'd reach for once a model is generated entirely from arrays rather than hand-written algebra.*
- **SciPy — `milp` reference:** <https://docs.scipy.org/doc/scipy/reference/generated/scipy.optimize.milp.html>
  *Why: SciPy's mixed-integer counterpart to `linprog`, referenced in Lecture 3 — worth knowing exists even though this week leans on PuLP for MIP work.*
- **HiGHS solver:** <https://highs.dev/>
  *Why: the modern, high-performance solver behind SciPy's `method="highs"` default — the actual engine doing the work when you call `linprog`.*

## Reference (keep in tabs)

- **NEOS Guide — Transportation Problem case study:** <https://neos-guide.org/case-studies/tp/>
  *Why: a second worked example of the exact model Lecture 2 builds, useful for cross-checking your own formulation instincts against an independent source.*
- **NEOS Guide — Mixed-Integer Linear Programming:** <https://neos-guide.org/case-studies/milp/>
  *Why: broader context on MIP applications beyond facility location — useful once Lecture 3's pattern feels familiar and you want to see where else it shows up.*
- **Google OR-Tools — Linear Optimization guide:** <https://developers.google.com/optimization/lp>
  *Why: a production-grade alternative to PuLP, worth a skim once this week's PuLP models feel comfortable — same modeling ideas, different library ergonomics, and the tool many real logistics teams reach for at larger scale.*
- **Wikipedia — Simplex algorithm:** <https://en.wikipedia.org/wiki/Simplex_algorithm>
  *Why: the mechanical procedure behind "walk from corner to corner" that Lecture 1 describes conceptually — read this if you want the full algorithmic detail.*
- **Wikipedia — Facility location problem:** <https://en.wikipedia.org/wiki/Facility_location_problem>
  *Why: the broader academic framing of Lecture 3's model, including variants (uncapacitated, p-median, p-center) this week doesn't cover.*
- **Dantzig, "Linear Programming and Extensions" (1963)** — the field's founding text. Available through most university libraries.
  *Why: for when "the simplex method walks between corners" stops being enough and you want the full proof machinery behind it.*

## Practice beyond the seed data

- **PuLP's own examples directory** (bundled with the library, or browsable on GitHub): <https://github.com/coin-or/pulp/tree/master/examples>
  *Why: a handful of classic LP/MIP formulations (blending, knapsack, assignment) worked end to end — good material for Homework Part A once you want more formulation reps.*
- **NEOS Server** (free, browser-based solvers for LP/MIP/nonlinear problems, no install required): <https://neos-server.org/neos/>
  *Why: a way to double-check a solved model against a completely independent solver implementation if you ever doubt a PuLP/CBC result.*

## Glossary

| Term | Definition |
|------|------------|
| **Decision variable** | A quantity the model is free to choose (how much to ship, whether to open a facility). |
| **Objective function** | The single linear expression being maximized or minimized. |
| **Constraint** | A linear (in)equality limiting the decision variables (capacity, demand, non-negativity). |
| **Feasible region** | The set of all points satisfying every constraint simultaneously. |
| **Corner point / vertex** | A point where enough constraints intersect to pin down a unique solution; LP optima always occur at one. |
| **Binding constraint** | A constraint that holds with equality at the optimum — the resource is fully used, zero slack. |
| **Slack** | The unused amount of a `≤` constraint's right-hand side at a given solution. |
| **Simplex method** | The classic algorithm that solves LPs by walking between adjacent corner points. |
| **Shadow price (dual value)** | The rate of change in the optimal objective value per unit change in a constraint's right-hand side. |
| **Reduced cost** | For a non-basic (unused) variable, how much its objective coefficient would need to improve before it became worth using. |
| **Transportation problem** | The classic multi-source, multi-destination minimum-cost flow model. |
| **Balanced network** | A transportation problem where total supply exactly equals total demand. |
| **Dummy source/sink** | An artificial node with a high cost, added to absorb a supply/demand imbalance so the model stays solvable. |
| **Mixed-integer program (MIP)** | An optimization problem where some variables are restricted to integer (often binary) values. |
| **Binary variable** | A variable restricted to exactly 0 or 1, typically representing an on/off or yes/no decision. |
| **LP relaxation** | The same MIP with integrality dropped, allowing fractional values — used as a bound and a diagnostic. |
| **Branch and bound** | The standard algorithm for solving MIPs: recursively split on a fractional variable, pruning branches that can't beat the best known integer solution. |
| **Facility location problem** | Deciding which of several candidate sites to open, trading fixed cost against variable service cost. |
| **Infeasible** | No point satisfies every constraint simultaneously — usually a modeling or data error, or a genuine real-world contradiction. |
| **Unbounded** | The objective can improve without limit — almost always means a real-world constraint is missing from the model. |
