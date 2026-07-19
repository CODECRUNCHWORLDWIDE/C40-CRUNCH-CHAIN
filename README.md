# C40 · Crunch Chain

> A free, open-source 12-week course to optimize supply chains through analytics, automation, and intelligent operations — demand forecasting, inventory, logistics, and optimization in Python on SQL data.

[![License: GPL v3](https://img.shields.io/badge/License-GPL%20v3-blue.svg)](LICENSE)
[![PostgreSQL · Python](https://img.shields.io/badge/stack-PostgreSQL_·_Python-2463EB.svg)](#stack)
[![Built in the open](https://img.shields.io/badge/built-in%20the%20open-2463EB.svg)](https://github.com/CODECRUNCHWORLDWIDE)

C40 is an applied operations course that takes you from "what is a supply chain?" to running a chain on numbers — forecasting demand, sizing inventory, routing freight, and solving a network with linear programming. Every workflow that a business would traditionally push into a spreadsheet is done here with **SQL (PostgreSQL 16) + Python (pandas)** instead, because a supply chain outgrows a spreadsheet the moment it has more than one warehouse. Pairs naturally with [C33 Crunch SQL](../C33-CRUNCH-SQL/) for the data layer and [C5 Crunch AI & Data Science](../C5-CRUNCH-AI-DATA-SCIENCE/) for the modeling depth.

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
