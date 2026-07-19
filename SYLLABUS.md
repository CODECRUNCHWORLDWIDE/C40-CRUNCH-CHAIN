# C40 · Crunch Chain — Syllabus

**Format:** 12 weeks · self-paced · open-source (GPL-3.0). Each week = 3 lectures, 3 exercises, 2 challenges, a mini-project, homework, and a quiz.

**Prerequisites:** You can run commands in a terminal and read basic Python. Comfort with SQL helps but is not required — [C33 Crunch SQL](../C33-CRUNCH-SQL/) is the ideal companion. No supply-chain background assumed; this course goes from foundations to expert-level optimization.

**Data rule:** all records live in **SQL (PostgreSQL 16, SQLite fallback)** and are analyzed in **Python (pandas)**. Spreadsheets are never used as a database — if a workflow would traditionally sit in Excel, we do it in SQL + Python and say why.

**Assessment (honor-based):** tick each lesson complete in the reader; finish the weekly mini-project and quiz. A certificate is issued on 100% completion to One-Stop members.

| Week | Theme | Mini-project |
|------|-------|--------------|
| 1 | Supply chain foundations, flows, KPIs, network design | Map a real chain end to end with its KPIs |
| 2 | SQL + pandas data foundations for operations | Model + load a multi-echelon network into Postgres |
| 3 | Demand forecasting fundamentals — baselines, seasonality, error | Forecast 50 SKUs and score MAPE/bias vs a naive baseline |
| 4 | Advanced forecasting — ML, hierarchical, backtesting | Backtest an ML forecast against classical models |
| 5 | Inventory — EOQ, reorder point, safety stock, service level | Set stocking policy for a catalog to a 95% service level |
| 6 | Procurement, spend analysis, supplier scorecards, lead-time risk | Build a supplier scorecard + spend cube in SQL |
| 7 | Logistics — lane cost, mode selection, routing | Solve a lowest-cost routing + mode plan |
| 8 | Warehousing — slotting, pick profiles, throughput, labor | Slot a warehouse from an order-line pick profile |
| 9 | Optimization — LP/MIP for sourcing + network flow (PuLP/SciPy) | Solve a plant-to-DC network flow at min cost |
| 10 | S&OP — reconciling demand, supply, capacity, finance | Run one S&OP cycle to a balanced plan |
| 11 | Risk, resilience, digital supply chain, automation, AI in ops | Stress-test a disruption + automate a replan pipeline |
| 12 | Capstone — profile + optimize a full network end to end | Cut total landed cost against a service constraint |

**Outcome:** forecast demand, size inventory, route freight, and optimize a supply-chain network end to end — running the whole chain on SQL data and Python, never a spreadsheet.
