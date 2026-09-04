# Week 9 — Optimization with Linear Programming

> **Goal:** by Sunday you can take a real sourcing or network decision — which plant should ship to which distribution center, which facilities should even exist — state it as decision variables, an objective, and constraints, hand it to a solver, and read back not just the answer but *why* it's the answer, in dollars per unit of capacity.

Welcome back to **C40 · Crunch Chain**. For eight weeks you've been analyzing a network someone else already built — forecasting demand against it, sizing inventory for it, scoring suppliers and carriers who feed it. This week you get to *design and route* it. Optimization is the mathematical machinery behind the sentence "ship this much from here to there, at minimum cost, without violating any constraint" — and it turns out that sentence covers an enormous share of what a supply-chain analyst is actually paid to do. We continue with **Crunch Gear**, the outdoor-apparel company you've followed since Week 1. This week you sit with its network-planning team as it decides how to flow product from three production sources to three regional distribution centers, and whether one of its own DCs is even worth keeping open.

**Data rule, unchanged:** every dataset this week — plant capacities, DC demand, lane costs, candidate-site economics — lives in **SQL (PostgreSQL 16, SQLite fallback)** and is modeled in **Python** with the solver libraries **PuLP** and **SciPy**. An optimization problem is not something you eyeball in a spreadsheet by trying a few combinations by hand; it is a precise mathematical object with a guaranteed-optimal answer, and that answer should come from a solver reading real rows out of a real table — never a spreadsheet "Goal Seek."

## Learning objectives

By the end of this week, you will be able to:

- **Formulate** an optimization problem from a plain-English business scenario: identify the decision variables, write the objective function, and state every constraint precisely — the hardest and most valuable skill in this week.
- **Solve** linear programs with **PuLP** and **SciPy** (`linprog`), and read a solution back correctly — which variables are basic, which constraints bind, and what the **shadow prices** (dual values) tell you about the marginal value of one more unit of capacity or demand.
- **Model and solve the classic transportation problem** — flow from multiple plants to multiple distribution centers at minimum total shipping cost — and interpret the result as a concrete shipping plan.
- **Build a facility-location model** with binary open/close decisions, recognizing when a problem needs **mixed-integer programming (MIP)** instead of pure LP, and why a fractional "open 60% of a warehouse" answer is meaningless.
- **Apply** everything above to production planning and capacitated sourcing — multi-period, multi-product problems where capacity, not demand, is the binding constraint.

## Standards this week meets

| Bar | What this week is measured against |
| --- | --- |
| University | `ISM 4400` — apply linear programming, the transportation model and facility location to an operations decision, and interpret the solution. |
| Industry | Deliver an optimisation as a pipeline a colleague or a scheduled job can re-run next month when the forecast changes, not as a notebook full of hard-coded numbers. |
| Beyond the bar | Shadow prices are read back off the solved model and written into the database alongside the shipping plan, so the marginal value of one more unit of capacity is part of the deliverable rather than a footnote — `mini-project/README.md` |

## Prerequisites

- Comfortable with `SELECT`, `WHERE`, `JOIN`, and basic aggregates in SQL (Weeks 1–8 of this course, or [C33 Crunch SQL](../../../C33-CRUNCH-SQL/)).
- Comfortable with basic Python: functions, loops, lists/dicts, and reading a table into a pandas DataFrame with `pandas.read_sql`.
- High-school algebra — solving two linear equations in two unknowns, plotting a line, shading a region. Lecture 1 rebuilds this from the geometry up; you don't need to have seen "linear programming" before.
- Python 3.10+ with **PuLP** and **SciPy** installed, plus PostgreSQL 16+ or SQLite 3.35+. See [`resources.md`](./resources.md) for install steps.

## Setup — install the solvers and seed the network

Install the two libraries this week runs on:

```bash
pip install pulp scipy pandas
```

PuLP ships with the open-source **CBC** solver built in — no separate solver install needed. Verify it in a Python shell:

```python
import pulp
print(pulp.listSolvers(onlyAvailable=True))   # should include 'PULP_CBC_CMD'
```

Everything in Lecture 2, Exercise 2, and the mini-project runs against one small network: **three production sources shipping to three distribution centers.** Seed it once, before Lecture 2.

**PostgreSQL:**

```bash
createdb crunch_network
psql crunch_network
```

**SQLite:**

```bash
sqlite3 crunch_network.db
```

Then paste this into the shell (identical on both engines):

```sql
CREATE TABLE plants (
    plant_id                TEXT PRIMARY KEY,
    plant_name               TEXT    NOT NULL,
    monthly_capacity_units   INTEGER NOT NULL
);

CREATE TABLE distribution_centers (
    dc_id                    TEXT PRIMARY KEY,
    dc_name                   TEXT    NOT NULL,
    monthly_demand_units      INTEGER NOT NULL
);

CREATE TABLE lane_costs (
    plant_id       TEXT    NOT NULL REFERENCES plants(plant_id),
    dc_id          TEXT    NOT NULL REFERENCES distribution_centers(dc_id),
    cost_per_unit  NUMERIC NOT NULL,
    PRIMARY KEY (plant_id, dc_id)
);

INSERT INTO plants VALUES
('EP',  'El Paso Plant',        4200),
('GDL', 'Guadalajara CMT',      5800),
('HCM', 'Ho Chi Minh CMT',      5000);

INSERT INTO distribution_centers VALUES
('AUS', 'Austin East DC',   6000),
('MEM', 'Memphis DC',       5000),
('RNO', 'Reno DC',          4000);

INSERT INTO lane_costs VALUES
('EP','AUS',4), ('EP','MEM',6), ('EP','RNO',5),
('GDL','AUS',3), ('GDL','MEM',7), ('GDL','RNO',8),
('HCM','AUS',9), ('HCM','MEM',8), ('HCM','RNO',6);
```

Sanity checks — total plant capacity and total DC demand should both print `15000`:

```sql
SELECT SUM(monthly_capacity_units) FROM plants;
SELECT SUM(monthly_demand_units) FROM distribution_centers;
```

This network is **balanced** on purpose (supply exactly equals demand) — it keeps the transportation problem in Lecture 2 clean while you learn the shape. Lecture 1 and Lecture 3 introduce their own small, self-contained datasets (a production-mix problem and a facility-location problem) inline in the lecture text — you don't need extra seed tables for those.

## Weekly schedule

Adds up to roughly the course's **~28 hr/week full-time pace**.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|--------------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | LP formulation, geometry, PuLP + SciPy basics | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | Transportation problem in PuLP; shadow prices | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Wednesday | Facility location, binary variables, MIP | 2h | 1.5h | 1h | 0.5h | 1h | 0h | 6h |
| Thursday | Multi-product network design (challenge) | 0h | 0h | 1.5h | 0.5h | 1h | 1h | 4h |
| Friday | Capacitated production plan (challenge); catch-up | 0h | 0h | 1.5h | 0.5h | 1h | 1.5h | 4.5h |
| Saturday | Mini-project — network flow, SQL in/out | 0h | 0h | 0h | 0h | 0h | 3h | 3h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **4h** | **4h** | **4h** | **3.5h** | **5h** | **5.5h** | **28h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-intro-to-linear-programming.md](./lecture-notes/01-intro-to-linear-programming.md) | Decision variables, objective, constraints; the geometry of LP; solving with PuLP and SciPy `linprog` | 2h |
| 2 | [lecture-notes/02-transportation-and-network-flow.md](./lecture-notes/02-transportation-and-network-flow.md) | The transportation problem; modeling plant→DC flow in PuLP; reading shadow prices | 2h |
| 3 | [lecture-notes/03-facility-location-and-mip.md](./lecture-notes/03-facility-location-and-mip.md) | Binary open/close decisions, capacitated facility location, why it needs MIP not LP | 2h |
| 4 | [exercises/exercise-01-solve-an-lp-with-pulp.md](./exercises/exercise-01-solve-an-lp-with-pulp.md) | Formulate and solve a production-mix LP with PuLP | 1h |
| 5 | [exercises/exercise-02-transportation-problem.md](./exercises/exercise-02-transportation-problem.md) | Solve the Week 9 plant→DC transportation problem and verify the shadow prices | 1.5h |
| 6 | [exercises/exercise-03-facility-location-model.md](./exercises/exercise-03-facility-location-model.md) | Build the facility-location MIP from Lecture 3 in PuLP | 1.5h |
| 7 | [challenges/challenge-01-multi-product-network-design.md](./challenges/challenge-01-multi-product-network-design.md) | Extend the network to two products sharing plant capacity | 1.5h |
| 8 | [challenges/challenge-02-capacitated-production-plan.md](./challenges/challenge-02-capacitated-production-plan.md) | Multi-period capacitated production planning with inventory carryover | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Solve the plant→DC network flow at minimum cost, reading from and writing to SQL | 3h |
| 10 | [homework.md](./homework.md) | Extra formulation and solving practice | 5h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official docs, solver references, tools to install | — |

## By the end of this week you can…

- Look at a sourcing or routing decision described in English and write down its decision variables, objective, and constraints without hand-waving.
- Solve an LP in PuLP or SciPy and correctly read which constraints bind, which don't, and what a shadow price means in plain business language.
- Build and solve the transportation problem for a real plant-to-DC network, and know it's the same underlying model whether you have 3 nodes or 300.
- Explain, with a worked example, why a facility-location decision needs binary variables and a MIP solver — and why rounding an LP's fractional answer is not the same thing.
- Pull a network's costs and capacities out of SQL, hand them to a solver, and write the optimal plan back into a table — the actual shape of this work in industry.

## Up next

Week 10 — S&OP: reconciling demand, supply, capacity, and finance into one balanced plan — this week's solver output becomes one input into that larger monthly cycle.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
