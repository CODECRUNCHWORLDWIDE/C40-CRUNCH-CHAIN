# Week 12 — Resources

Free, public, no signup unless noted. Read the "required" set before Lecture 2; treat the rest as reference you dip into when a specific question comes up.

## Install first

- **PostgreSQL 16+** — the course's primary engine: <https://www.postgresql.org/download/> · macOS: [Postgres.app](https://postgresapp.com/). Linux: `sudo apt install postgresql` / `sudo dnf install postgresql-server`. Windows: the EDB installer.
- **SQLite 3.35+** — the zero-setup fallback: <https://www.sqlite.org/download.html>. Check with `sqlite3 --version`.
- **Python 3.10+** with this week's full stack:
  ```bash
  pip install pandas numpy scipy sqlalchemy pulp
  ```
  `pulp` is new this week — it's a pure-Python LP/MIP modeling library that ships with the free, open-source **CBC** solver bundled in, so `pip install pulp` is genuinely all you need; no separate solver installation or license.
- **(Optional) matplotlib**, for the waterfall/monthly-cost charts in Lecture 3 and Challenge 1: `pip install matplotlib`.
- **(Optional) SciPy's `linprog`** as an alternative to PuLP if you prefer array-based LP formulation over PuLP's expression-based syntax — already installed via `scipy` above.

## Required reading (this week's core)

- **PuLP documentation — "A Blending Problem" and "The Optimal Diet Problem" tutorials:** <https://coin-or.github.io/pulp/CaseStudies/index.html>
  *Why: PuLP's own worked examples are the fastest way to see the pattern — decision variables, objective, constraints — used identically in this week's network-flow LP.*
- **SciPy — `scipy.optimize.linprog` documentation:** <https://docs.scipy.org/doc/scipy/reference/generated/scipy.optimize.linprog.html>
  *Why: the array-based alternative to PuLP; useful to see the exact same transportation problem expressed as matrices, which clarifies what PuLP is doing under the hood.*
- **Investopedia — "Economic Order Quantity (EOQ)":** <https://www.investopedia.com/terms/e/economicorderquantity.asp>
  *Why: a clean plain-language refresher on the EOQ formula this week reuses from Week 5, with the trade-off (ordering cost vs. holding cost) stated in business terms, not just algebra.*
- **NIST/SEMATECH e-Handbook of Statistical Methods — "What are Percentiles?" (for the Z-score / normal distribution used in safety stock):** <https://www.itl.nist.gov/div898/handbook/eda/section3/eda362.htm>
  *Why: the reference behind `Z = 1.645` at 95%, `Z = 2.326` at 99%, and why that value climbs faster than linearly as your target service level approaches 100% (Homework Problem 3).*

## LP/MIP — going deeper

- **Google OR-Tools — Linear Optimization overview:** <https://developers.google.com/optimization/lp>
  *Why: a production-grade alternative to PuLP, used at real scale in industry; useful context for "what does this look like when the network has thousands of lanes, not twelve."*
- **COIN-OR CBC solver documentation:** <https://github.com/coin-or/Cbc>
  *Why: the actual solver PuLP calls by default — useful if a solve ever times out or returns an unexpected status and you want to understand what's happening underneath `prob.solve()`.*
- **Winston, W.L., *Operations Research: Applications and Algorithms* — Transportation and Assignment Problems chapter.** Available through most university library systems.
  *Why: the classic textbook treatment of exactly the transportation-problem structure this week's network-flow LP is built on, including the stepping-stone method used to verify Lecture 1's hand-worked solution.*

## Presentation and communication

- **Barbara Minto's Pyramid Principle (summary article, McKinsey alumni network):** search "Minto Pyramid Principle summary" — widely summarized freely online, original book is not free.
  *Why: the "lead with the conclusion, support it after" structure behind Lecture 3's memo template, from the person who arguably invented that discipline for management consulting.*
- **`dataviz` skill (this environment)** — if you're working inside an agent environment with this skill installed, invoke it before building the waterfall chart from Lecture 3 §4.
  *Why: chart color, form, and layout guidance consistent with the "one chart, chosen on purpose" principle this week teaches.*

## Practice beyond the seed data

- **Google OR-Tools examples (Python, free, open source):** <https://developers.google.com/optimization/lp> includes runnable transportation and assignment problem code you can compare against your own Lecture 2 §4 implementation.
- **PuLP's GitHub examples directory:** <https://github.com/coin-or/pulp/tree/master/examples>
  *Why: more worked LP formulations, useful for the mini-project's multi-echelon stretch goal.*

## Glossary

| Term | Definition |
|------|------------|
| **Multi-echelon network** | A supply chain with more than one tier of intermediate nodes between raw material and customer (here: plants → DCs → regions). |
| **Total landed cost** | Every cost to get a unit from source to customer: production, inbound freight, outbound freight, DC fixed cost, and inventory holding cost. |
| **Scoping document** | A short, explicit statement of a project's objective, in-scope levers, held-fixed assumptions, constraint, and target — written before any modeling starts. |
| **Linear program (LP)** | An optimization model with a linear objective function and linear constraints over continuous decision variables. |
| **Decision variable** | A quantity the model is free to choose (here: how many units flow on each DC→region lane). |
| **Objective function** | The linear expression an LP minimizes or maximizes — here, total outbound freight cost. |
| **Constraint** | A linear equation or inequality the solution must satisfy — here, demand-satisfaction (`==`) and capacity (`<=`). |
| **Feasible / Infeasible** | A solution (or model) that does / does not satisfy every constraint simultaneously. |
| **Optimal** | The best feasible solution per the objective function — the status PuLP reports when the solver succeeds. |
| **Shadow price (dual value)** | The marginal value of relaxing a binding constraint by one unit — how much the objective would improve if a limit were loosened slightly. |
| **Transportation problem** | The classic LP structure of shipping from multiple supply points to multiple demand points at minimum total cost, subject to supply and demand limits. |
| **Cycle service level** | The probability that demand during a single replenishment lead time does not exceed on-hand inventory (i.e., no stockout that cycle). |
| **Backtesting** | Testing a forecasting method against historical data it wasn't trained on, to estimate real-world forecast error before trusting it. |
| **MAPE (Mean Absolute Percentage Error)** | Average of `|forecast − actual| / actual` across all forecasted periods, expressed as a percentage. |
| **Waterfall chart** | A chart showing how a starting value changes through a sequence of additions and subtractions to reach an ending value — ideal for "baseline minus lever A minus lever B equals optimized total." |

---

*Broken link? Open an issue or PR.*
