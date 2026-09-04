# C40 · Crunch Chain

> A free, open-source 12-week course to optimize supply chains through analytics, automation, and intelligent operations — demand forecasting, inventory, logistics, and optimization in Python on SQL data.

[![License: GPL v3](https://img.shields.io/badge/License-GPL%20v3-blue.svg)](LICENSE)
[![PostgreSQL · Python](https://img.shields.io/badge/stack-PostgreSQL_·_Python-2463EB.svg)](#stack)
[![Built in the open](https://img.shields.io/badge/built-in%20the%20open-2463EB.svg)](https://github.com/CODECRUNCHWORLDWIDE)

C40 is an applied operations course that takes you from "what is a supply chain?" to running a chain on numbers — forecasting demand, sizing inventory, routing freight, and solving a network with linear programming. Every workflow that a business would traditionally push into a spreadsheet is done here with **SQL (PostgreSQL 16) + Python (pandas)** instead, because a supply chain outgrows a spreadsheet the moment it has more than one warehouse. Pairs naturally with [C33 Crunch SQL](../C33-CRUNCH-SQL/) for the data layer and [C5 Crunch AI & Data Science](../C5-CRUNCH-AI-DATA-SCIENCE/) for the modeling depth.

---

## Standards & equivalency

> C40 stands in for a university's operations and supply chain management course.

**University equivalent.** Operations and Supply Chain Management — `ISM 4400`, `MAN 3025`, `SCM 3301`. Coverage: full. Every outcome in the table below is taught here and assigned as work — an exercise, a challenge, homework, a quiz item or the capstone — not merely mentioned in a lecture.

C40 carries no credit, no transcript entry, no accreditation and no proctored exam. The equivalence is one of **content and skill**: the outcomes below are taught at the same depth or deeper, and every one of them is assessed. What a registrar records is not something an open repository can give you.

| University outcome | Where this course teaches it | Depth |
| --- | --- | --- |
| Describe a supply chain end to end — nodes, echelons, and the flows of product, information and cash moving through it | [Week 01](curriculum/week-01-supply-chain-foundations-and-network-design/) | same |
| Define and compute the operating measures by which a chain is judged — fill rate, on-time-in-full, perfect order, inventory turns, cash-to-cash cycle | [Week 01](curriculum/week-01-supply-chain-foundations-and-network-design/) | same |
| Reason about network structure: echelon count, centralised versus distributed facilities, and the cost-service trade-off that governs the choice | [Week 01](curriculum/week-01-supply-chain-foundations-and-network-design/) | same |
| Work with operating records — orders, shipments, receipts, inventory — and produce a performance report from them | [Week 02](curriculum/week-02-sql-and-pandas-data-foundations/) | deeper |
| Forecast demand with time-series methods, and measure forecast error against a naive benchmark | [Week 03](curriculum/week-03-demand-forecasting-fundamentals/) | same |
| Handle seasonality, causal drivers and irregular demand, and validate a forecast on data it has not seen | [Week 04](curriculum/week-04-advanced-forecasting-and-ml/) | deeper |
| Determine order quantity from the trade-off between holding and ordering cost, using the economic order quantity model | [Week 05](curriculum/week-05-inventory-management-eoq-and-safety-stock/) | same |
| Set safety stock and reorder points to a stated service level under demand and lead-time uncertainty, and choose a review policy | [Week 05](curriculum/week-05-inventory-management-eoq-and-safety-stock/) | same |
| Solve the single-period stocking decision where there is no next cycle to recover in — the newsvendor problem | [Week 05](curriculum/week-05-inventory-management-eoq-and-safety-stock/) | same |
| Analyse purchasing spend and evaluate suppliers on cost, quality, delivery and lead-time reliability | [Week 06](curriculum/week-06-procurement-and-supplier-analytics/) | deeper |
| Source on total cost of ownership rather than unit price, and defend a sourcing recommendation | [Week 06](curriculum/week-06-procurement-and-supplier-analytics/) | same |
| Select transportation modes and carriers on cost, speed and reliability, and analyse freight spend and delivery performance | [Week 07](curriculum/week-07-logistics-and-transportation-analytics/) | same |
| Plan distribution routes under a vehicle capacity constraint | [Week 07](curriculum/week-07-logistics-and-transportation-analytics/) | deeper |
| Describe warehouse operations from receiving to shipping, and design storage, picking and labour to a throughput target | [Week 08](curriculum/week-08-warehousing-and-fulfillment/) | same |
| Apply quantitative models — linear programming, the transportation model, facility location — to an operations decision, and interpret the solution and its shadow prices | [Week 09](curriculum/week-09-optimization-with-linear-programming/) | deeper |
| Perform aggregate planning: compare chase, level and mixed strategies on total cost against capacity | [Week 10](curriculum/week-10-sales-and-operations-planning/) | same |
| Run a sales and operations planning cycle that reconciles demand, supply and the financial plan onto one set of numbers | [Week 10](curriculum/week-10-sales-and-operations-planning/) | same |
| Identify supply chain risk and single points of failure, prioritise them, and design mitigation whose cost you can state | [Week 11](curriculum/week-11-risk-resilience-and-the-digital-supply-chain/) | same |
| Explain the role of information systems and data in a modern operations function | [Week 11](curriculum/week-11-risk-resilience-and-the-digital-supply-chain/) | deeper |
| Bring the whole of the above onto one operating problem and defend the recommendation to management | [Week 12](curriculum/week-12-capstone-optimize-the-chain-end-to-end/) | deeper |

**The industry bar.** What an employer expects of somebody paid to run operations analytics, and where this course makes the learner do it. Two rows below say plainly that C40 does not ship the artefact the bar names, and what it uses instead — that is the honest position, not an oversight.

| What the job expects | Where this course does it |
| --- | --- |
| Work lands as a commit in a repository you own, not a file on your desktop | every exercise, challenge and mini-project ends with a `## Submission` line naming the file and the folder — for example [`curriculum/week-05-inventory-management-eoq-and-safety-stock/exercises/exercise-01-compute-eoq.md`](curriculum/week-05-inventory-management-eoq-and-safety-stock/exercises/exercise-01-compute-eoq.md) |
| You read work you did not produce and form a judgement on it | [`curriculum/week-04-advanced-forecasting-and-ml/homework.md`](curriculum/week-04-advanced-forecasting-and-ml/homework.md) — Problem 3 hands you a colleague's regression coefficient table to review before it ships, and Problem 4 hands you three backtest designs and asks which of them leak and exactly how |
| Results are checked, not trusted | C40 ships **no test suite and no `pytest` harness** — its deliverables are queries, models and written recommendations rather than a library. What it ships instead: published spot-check numbers on every guided exercise, the same figure computed twice in SQL and in pandas and reconciled, and validation assertions the learner is required to write into the code at [`curriculum/week-07-logistics-and-transportation-analytics/challenges/challenge-01-savings-algorithm-routing.md`](curriculum/week-07-logistics-and-transportation-analytics/challenges/challenge-01-savings-algorithm-routing.md) |
| Failure is taught from something real, not from a warning in prose | C40 carries **no `Common bugs to catch` section quoting a captured traceback**. Failure is taught instead as planted, findable error: a deliberate SQL-versus-pandas discrepancy at [`curriculum/week-02-sql-and-pandas-data-foundations/challenges/challenge-02-reconcile-sql-and-pandas-kpis.md`](curriculum/week-02-sql-and-pandas-data-foundations/challenges/challenge-02-reconcile-sql-and-pandas-kpis.md), leaking backtests to be caught in Week 04's homework, and a solver that must report `Infeasible` rather than a quietly wrong number in [`curriculum/week-12-capstone-optimize-the-chain-end-to-end/challenges/challenge-01-end-to-end-optimization.md`](curriculum/week-12-capstone-optimize-the-chain-end-to-end/challenges/challenge-01-end-to-end-optimization.md) |
| The system of record is a database, not a spreadsheet | [`curriculum/week-02-sql-and-pandas-data-foundations/exercises/exercise-01-seed-the-network-database.md`](curriculum/week-02-sql-and-pandas-data-foundations/exercises/exercise-01-seed-the-network-database.md) — a nine-table schema with keys and constraints, loaded and verified by the learner, and used by every week after it |
| Analysis re-runs when new data lands, without hand-editing | [`curriculum/week-09-optimization-with-linear-programming/mini-project/README.md`](curriculum/week-09-optimization-with-linear-programming/mini-project/README.md) — data in, solve, data back to SQL, re-runnable next month without touching the code |
| The output is portfolio-grade: a stranger can read it and know what you can do | [`curriculum/week-12-capstone-optimize-the-chain-end-to-end/mini-project/README.md`](curriculum/week-12-capstone-optimize-the-chain-end-to-end/mini-project/README.md), delivered as a one-page executive recommendation at [`curriculum/week-12-capstone-optimize-the-chain-end-to-end/challenges/challenge-02-executive-recommendation.md`](curriculum/week-12-capstone-optimize-the-chain-end-to-end/challenges/challenge-02-executive-recommendation.md) |
| The professional task is named, not implied | the `Standards this week meets` block in each of the twelve week READMEs |

**Beyond both bars.** Clearing the two floors is entry, not success. Open any of these and check it in under a minute.

| What we add | Which bar it beats | Where it lives |
| --- | --- | --- |
| Every week's quiz publishes a worked answer key on the same page — the letter and the arithmetic behind it — folded so you can attempt first, but never withheld until a deadline | both | [`curriculum/week-05-inventory-management-eoq-and-safety-stock/quiz.md`](curriculum/week-05-inventory-management-eoq-and-safety-stock/quiz.md) |
| Every guided exercise publishes the numbers your work has to land on, so you can prove yourself right or wrong without waiting for a grader | both | [`curriculum/week-05-inventory-management-eoq-and-safety-stock/exercises/exercise-01-compute-eoq.md`](curriculum/week-05-inventory-management-eoq-and-safety-stock/exercises/exercise-01-compute-eoq.md) |
| Every open-ended challenge publishes the rubric it is judged against before you start, row by row, with the weak answer written out beside the strong one | university | [`curriculum/week-07-logistics-and-transportation-analytics/challenges/challenge-01-savings-algorithm-routing.md`](curriculum/week-07-logistics-and-transportation-analytics/challenges/challenge-01-savings-algorithm-routing.md) |
| Optimisation is written and solved by the learner, not described — a transportation model, a facility-location model with binary open/close decisions, and the shadow prices read back off the solution | university | [`curriculum/week-09-optimization-with-linear-programming/exercises/exercise-01-solve-an-lp-with-pulp.md`](curriculum/week-09-optimization-with-linear-programming/exercises/exercise-01-solve-an-lp-with-pulp.md) |
| Every figure in the course comes out of a seeded database the learner loads and verifies, and every KPI is computed twice — once in SQL, once in pandas — and reconciled | industry | [`curriculum/week-02-sql-and-pandas-data-foundations/challenges/challenge-02-reconcile-sql-and-pandas-kpis.md`](curriculum/week-02-sql-and-pandas-data-foundations/challenges/challenge-02-reconcile-sql-and-pandas-kpis.md) |
| A monitoring week that a management syllabus does not reach: rolling baselines over operating data, an anomaly detector whose threshold is chosen by costing both kinds of mistake, and the automated plan-forecast-replan loop around it | both | [`curriculum/week-11-risk-resilience-and-the-digital-supply-chain/exercises/exercise-03-automate-a-daily-ops-pipeline.md`](curriculum/week-11-risk-resilience-and-the-digital-supply-chain/exercises/exercise-03-automate-a-daily-ops-pipeline.md) |
| The learner finishes holding a public repository somebody can clone and re-run — a forecast, a stocking policy, a solved network and a memo — instead of a grade only a registrar can see | both | [`curriculum/week-12-capstone-optimize-the-chain-end-to-end/mini-project/README.md`](curriculum/week-12-capstone-optimize-the-chain-end-to-end/mini-project/README.md) |

**Gaps we declare.** None against the outcome set above. Three things sit outside the claim and C40 does not make them: it does not teach quality management (statistical process control, six sigma), lean and just-in-time production, or materials requirements planning — where an operations section carries those alongside the supply chain half, C40 does not stand in for that half. It also ships no automated test suite, no continuous-integration pipeline and no linter configuration; the industry table above says what it uses in their place rather than implying they are there.

---

## Pathway summary

- **Full-time:** 12 weeks · ~28 hrs/week · ~336 hours
- **Working-analyst pace:** 6 months · ~14 hrs/week
- **Evening pace:** 12 months · ~7 hrs/week

See [`SYLLABUS.md`](SYLLABUS.md).

---

## What you will be able to do at the end of 12 weeks

- **Model a supply chain as data:** design the schema for a multi-echelon network — suppliers, plants, warehouses, lanes, SKUs, orders — and query it fluently in SQL and pandas.
- **Forecast demand you can defend:** build, backtest, and error-score baseline, seasonal, and ML forecasts; know when a simple model beats a fancy one.
- **Size inventory on purpose:** compute EOQ, reorder points, and safety stock to a target service level, and see the cost of getting it wrong.
- **Run procurement on evidence:** score suppliers, analyze spend, and model lead-time risk instead of guessing.
- **Move freight efficiently:** analyze lanes, model transportation cost, and solve routing and mode-selection trade-offs.
- **Design fulfillment that flows:** slot a warehouse, size labor to a pick profile, and measure throughput and order cycle time.
- **Optimize with real math:** formulate and solve linear and integer programs for network design, sourcing, and transportation with PuLP/SciPy.
- **Plan the whole business:** run an S&OP cycle that reconciles demand, supply, and finance on one set of numbers.
- **Make the chain resilient and intelligent:** stress-test for disruption, and automate the plan-forecast-replan loop with AI and pipelines.
- **Optimize a chain end to end:** take a messy, realistic network and drive its total cost down against a service constraint — the capstone.

---

## Curriculum (12 weeks)

| Week | Topic | You leave able to… |
|------|-------|--------------------|
| 1 | [Supply chain foundations & network design](curriculum/week-01-supply-chain-foundations-and-network-design/) | Map a chain end to end and reason about its network shape and KPIs. |
| 2 | [SQL & pandas data foundations](curriculum/week-02-sql-and-pandas-data-foundations/) | Model and query a supply-chain dataset in Postgres and pandas. |
| 3 | [Demand forecasting fundamentals](curriculum/week-03-demand-forecasting-fundamentals/) | Build baseline + seasonal forecasts and score their error honestly. |
| 4 | [Advanced forecasting & ML](curriculum/week-04-advanced-forecasting-and-ml/) | Backtest ML and hierarchical forecasts; pick the model that wins. |
| 5 | [Inventory management — EOQ & safety stock](curriculum/week-05-inventory-management-eoq-and-safety-stock/) | Set order quantity, reorder point, and safety stock to a service level. |
| 6 | [Procurement & supplier analytics](curriculum/week-06-procurement-and-supplier-analytics/) | Analyze spend and score suppliers on cost, quality, and lead time. |
| 7 | [Logistics & transportation analytics](curriculum/week-07-logistics-and-transportation-analytics/) | Model lane cost and solve mode + routing trade-offs. |
| 8 | [Warehousing & fulfillment](curriculum/week-08-warehousing-and-fulfillment/) | Slot a warehouse and size fulfillment to a pick profile. |
| 9 | [Optimization with linear programming](curriculum/week-09-optimization-with-linear-programming/) | Formulate and solve LP/MIP for sourcing and network flow. |
| 10 | [Sales & operations planning](curriculum/week-10-sales-and-operations-planning/) | Run an S&OP cycle reconciling demand, supply, and finance. |
| 11 | [Risk, resilience & the digital supply chain](curriculum/week-11-risk-resilience-and-the-digital-supply-chain/) | Stress-test disruptions and automate the plan-replan loop with AI. |
| 12 | [Capstone — optimize the chain end to end](curriculum/week-12-capstone-optimize-the-chain-end-to-end/) | Cut total cost against a service target on a full network. |

---

## How to navigate a week

Every week folder holds the same structure:

- **`README.md`** — the week overview + how the pieces fit + the week's goal.
- **`lecture-notes/`** — 3 lectures (~2 hrs each), the conceptual core.
- **`exercises/`** — 3 short, guided reps against a real dataset.
- **`challenges/`** — 2 open-ended problems with no single right answer.
- **`mini-project/`** — one build that ties the week together.
- **`homework.md`**, **`quiz.md`**, **`resources.md`** — practice, self-check, and further reading.

---

## Stack

PostgreSQL (16+) as the system of record for all supply-chain data, with SQLite for zero-setup practice, and Python (pandas, NumPy, statsmodels, scikit-learn, PuLP/SciPy) for forecasting and optimization. Everything is free and runs on macOS, Linux, and Windows. **No spreadsheets are used as a data store** — records live in SQL, analysis runs in pandas. A seed dataset (a small multi-echelon network with orders and shipments) ships with the course.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · [Browse all courses](https://codecrunchglobal.vercel.app/courses)*
