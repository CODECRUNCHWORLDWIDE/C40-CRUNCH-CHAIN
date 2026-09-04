# Week 12 — Capstone — Optimize the Chain End to End

> **Goal:** by Sunday you have built, run, and defended one reproducible pipeline that assembles a full multi-echelon network in SQL, forecasts demand, sizes inventory policy, optimizes the flow of product through the network with linear programming, and prints an executive-ready recommendation — and you can quote, from memory, the one number that matters: how much total network cost you cut, and what you had to hold constant to earn it.

Welcome to the last week of **C40 · Crunch Chain**. Every week before this one taught you one piece of the job — map the network (Week 1), model it in SQL (Week 2), forecast demand (Weeks 3–4), size inventory (Week 5), evaluate suppliers (Week 6), route freight (Week 7), and (per the syllabus) slot a warehouse, solve network flow with LP/MIP, run an S&OP cycle, and stress-test for risk (Weeks 8–11). This week you stop learning pieces and **ship the whole thing**: one network, one pipeline, one number, one recommendation — the exact shape of a real capstone project on a real operations analytics team.

We use one running network all week: **Crunch Gear's Alpine jacket line** — two supply sources (an overseas contract mill and a domestic cut-and-sew plant), three distribution centers (the same Austin East, Memphis DC, and Reno DC you've seen since Week 1), four demand regions, and six SKUs. It is realistic enough that a real total-landed-cost analysis on it teaches the real skill, and small enough that you can hand-verify every number the code produces — which you will do at least once this week, because an operations analyst who can't sanity-check a solver's output by hand is an operations analyst nobody trusts with a seven-figure recommendation.

**Data rule for this course, one more time:** every table in this capstone — the network, the demand history, the forecast, the inventory policy, the optimized flow plan — lives in **SQL (PostgreSQL 16, SQLite fallback)** and is analyzed with **Python (pandas, PuLP/SciPy)**. Not one spreadsheet touches this project. If you've ever seen a "network optimization" done in Excel Solver, you already know why: it doesn't scale past a few dozen decision variables, it has no audit trail, and it can't be re-run automatically when next month's data lands. This capstone's whole point is a pipeline you can re-run on command.

## Learning objectives

By the end of this week, you will be able to:

- **Assemble** a full multi-echelon network dataset in SQL — plants, DCs, regions, SKUs, lanes, costs, capacities, and demand history — and **profile** its current total cost and service level as a documented, defensible baseline.
- **Integrate** demand forecasting, inventory policy (EOQ, safety stock, reorder point), and network flow optimization into one reproducible pipeline, where each stage's output is the next stage's input.
- **Formulate and solve** a linear program that minimizes total network cost subject to demand-satisfaction and DC-capacity constraints, using PuLP or SciPy.
- **Drive total landed cost down** against an explicit service-level constraint, and **quantify** exactly which lever — network reallocation, safety-stock right-sizing, or both — produced the savings.
- **Automate** the forecast → policy → optimize → report loop into a single script that anyone on the team can re-run when new data lands.
- **Present** the result as a one-page executive recommendation: the headline number, the trade-off, the ask, and the risks — the way you'd actually hand this to a VP of Supply Chain, not the way you'd hand it to another analyst.

## Standards this week meets

| Bar | What this week is measured against |
| --- | --- |
| University | `ISM 4400` — bring the whole operations and supply chain outcome set onto one operating problem, and defend the recommendation to management. |
| Industry | Cut total network cost against an explicit service constraint, and get the result past the ninety seconds of attention a VP gives a memo before forwarding it or dropping it. |
| Beyond the bar | The pipeline has to re-run end to end with no manual steps, and has to report `Infeasible` when a stress scenario genuinely cannot be met rather than returning a quietly wrong number — `challenges/challenge-01-end-to-end-optimization.md` |

## Prerequisites

- Comfortable with SQL joins, aggregation, and window functions (Weeks 1–2, and C33 Crunch SQL if you took it).
- Comfortable writing a forecast (naive/seasonal-naive, moving average, or exponential smoothing) and computing forecast error (MAE, MAPE) in Python — Weeks 3–4's territory.
- Know the EOQ and safety-stock formulas — `EOQ = sqrt(2DS/H)`, `SS = z·σ_LT`, `ROP = d̄·LT + SS` — Week 5's territory. This week reuses them; it does not re-derive them.
- **New this week:** basic linear programming with **PuLP** (`pip install pulp`) or **SciPy's `linprog`** (`pip install scipy`) — a decision variable, an objective function, and a handful of constraints. If you have never written an LP before, read [Resources](./resources.md) first; Lecture 2 also builds one from zero, slowly.
- PostgreSQL 16+ or SQLite 3.35+, and Python 3.10+ with `pandas`, `numpy`, and `pulp` installed.

## Setup — build the capstone network once

Every lecture, exercise, challenge, and the mini-project runs against the **same** network. Build it once, in a fresh database, before you open Lecture 1.

```bash
createdb crunch_chain_capstone           # PostgreSQL
psql crunch_chain_capstone
# — or —
sqlite3 crunch_chain_capstone.db         # SQLite fallback
```

Lecture 1 walks through the schema and every `CREATE TABLE`/`INSERT` statement in full — plants, DCs, regions, SKUs, lanes, costs, capacities — plus the Python demand-history generator that seeds 24 months of realistic, seasonal, reproducible monthly demand (seeded with `np.random.seed(40)`, so your numbers match everyone else's). Do not skip ahead to the exercises without running Lecture 1's setup block; every later file assumes this exact schema and these exact tables exist.

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-capstone-scoping-and-baseline.md](./lecture-notes/01-capstone-scoping-and-baseline.md) | Scoping the capstone, building the full network schema in SQL, generating 24 months of demand, profiling the naive baseline's cost and service | 2.5h |
| 2 | [lecture-notes/02-integrating-forecast-inventory-and-network.md](./lecture-notes/02-integrating-forecast-inventory-and-network.md) | Chaining forecast → inventory policy → network LP into one pipeline; formulating and solving the transportation LP with PuLP | 2.5h |
| 3 | [lecture-notes/03-presenting-operations-results.md](./lecture-notes/03-presenting-operations-results.md) | Turning solver output into an executive one-pager — headline number, trade-off, the ask, risks | 2h |
| 4 | [exercises/exercise-01-assemble-the-data-model.md](./exercises/exercise-01-assemble-the-data-model.md) | Build and verify the full schema and seed data | 1.5h |
| 5 | [exercises/exercise-02-run-forecast-and-policy.md](./exercises/exercise-02-run-forecast-and-policy.md) | Forecast every SKU-region pair; compute EOQ/safety stock/ROP for each | 2h |
| 6 | [exercises/exercise-03-optimize-and-report.md](./exercises/exercise-03-optimize-and-report.md) | Build and solve the network LP; produce a cost/service comparison report | 2h |
| 7 | [challenges/challenge-01-end-to-end-optimization.md](./challenges/challenge-01-end-to-end-optimization.md) | Run the full pipeline end to end on 24 months of data; hit the cost target under the service constraint; sensitivity-test it | 2.5h |
| 8 | [challenges/challenge-02-executive-recommendation.md](./challenges/challenge-02-executive-recommendation.md) | Turn your challenge-1 results into a one-page memo and defend it against three tough follow-up questions | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | The capstone deliverable: design, forecast, optimize, and automate the full chain end to end, with an executive report | 6h |
| 10 | [homework.md](./homework.md) | Extra practice — sensitivity analysis, alternate scenarios, a second network variant | 4h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | PuLP/SciPy docs, LP modeling references, presentation guides, install steps | — |

## Weekly schedule

Adds up to roughly the course's **~28 hr/week full-time pace**.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|----------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Scope the capstone; build the network + baseline | 2.5h | 1.5h | 0h | 0.5h | 0h | 0h | 4.5h |
| Tuesday | Forecast + inventory policy for every SKU-region | 0h | 2h | 0h | 0.5h | 1h | 0h | 3.5h |
| Wednesday | Formulate and solve the network LP | 2.5h | 2h | 0h | 0.5h | 1h | 0h | 6h |
| Thursday | End-to-end run + sensitivity testing | 0h | 0h | 2.5h | 0.5h | 1h | 1h | 5h |
| Friday | Present results; executive memo | 2h | 0h | 1.5h | 0.5h | 1h | 1h | 6h |
| Saturday | Mini-project | 0h | 0h | 0h | 0h | 0h | 3.5h | 3.5h |
| Sunday | Quiz + final review | 0h | 0h | 0h | 1h | 0h | 0.5h | 1.5h |
| **Total** | | **7h** | **5.5h** | **4h** | **3.5h** | **4h** | **6h** | **~30h** |

## By the end of this week you can…

- Stand up a full multi-echelon network — plants, DCs, regions, SKUs, lanes, capacities, and 24 months of demand — in SQL, from nothing, in under an hour.
- Chain a forecast into a safety-stock calculation into a network flow LP, without manually re-typing a single number between stages.
- Write and solve a transportation-style LP with PuLP that minimizes total cost subject to demand and capacity constraints, and explain in plain English what every constraint does.
- State, precisely, how much cost your recommendation saves, which lever earned which share of it, and what service-level guarantee it doesn't break.
- Write a one-page executive memo that a VP would actually read to the end.

## Course complete

This is the last week of **C40 · Crunch Chain**. If you've pushed all twelve mini-projects and this week's capstone, you've forecast demand, sized inventory, evaluated suppliers, routed freight, slotted a warehouse, optimized a network with linear programming, run an S&OP cycle, and stress-tested for risk — on real SQL tables and real Python code, never once on a spreadsheet acting as a database. That's the whole job. Go build the real thing.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
