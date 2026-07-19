# Week 11 — Risk, Resilience & the Digital Supply Chain

> **Goal:** by Sunday you can score every node and lane in a network for risk, simulate a real disruption end to end — cost, service damage, and recovery time — design a resilience response with numbers behind it, and stand up an automated daily pipeline that flags an operational anomaly before a human notices the trend on a report.

Welcome back to **C40 · Crunch Chain**. Every week so far has assumed the network works as planned: forecasts land close enough, suppliers ship on time, trucks arrive, warehouses flow. Week 11 is the week that assumption breaks on purpose. A fire at a contract manufacturer, a port closed by a typhoon, a ransomware outage on the order-management system — none of these are exotic. They are the ordinary cost of running a physical network that touches the real world, and a supply chain analyst's job is not to pretend they won't happen but to know **where** they're most likely to hit, **how much** they'll cost when they do, and **how fast** the network gets back to normal.

This week runs on two datasets, both new. A **16-row `network_risk_register`** scores every major single point of failure in Crunch Gear's network — DCs, suppliers, carriers, systems, even a concentrated wholesale account — on likelihood and impact. And a **180-row `daily_ops` table**, 60 days of daily operating metrics (orders, OTIF%, fill rate, lead time) across all three DCs, with a real disruption baked into the middle of it: on **April 15, 2026**, a fire shuts down **Andes Stitch Works**, the sole cut-make-trim (CMT) line for Crunch Gear's flagship Outerwear style (you met this supplier in [Week 6](../week-06-procurement-and-supplier-analytics/)). You'll watch Austin East's service levels collapse as the safety-stock buffer runs out, bottom out, and recover — and you'll build the tooling that would have caught it early.

**Data rule for this course:** every risk score, every disruption simulation, every anomaly-detection query this week runs in **SQL (PostgreSQL 16, SQLite fallback)** and/or **Python (pandas)** — never a spreadsheet. A risk register that lives in a shared Excel file is exactly the kind of "control tower" that silently goes stale; a risk register that lives in a table with a `mitigation_status` column and a `last_reviewed` process is something you can actually query, alert on, and trust.

## Learning objectives

By the end of this week, you will be able to:

- **Identify** supply chain risks and single points of failure (SPOFs) across a real multi-echelon network — facilities, suppliers, carriers, systems, and demand concentration — not just the obvious ones.
- **Score and prioritize** risk on a likelihood × impact matrix, defend a "must mitigate first" shortlist, and know the matrix's limitations (it's a prioritization tool, not a prediction).
- **Simulate a disruption** against real operating data — measure its cost in dollars and its damage in service-level terms, and measure recovery time precisely instead of by feel.
- **Design resilience** — buffers (safety stock), dual-sourcing, and network redundancy — and quantify the trade-off between the cost of resilience and the expected annual loss it removes.
- **Automate the plan-forecast-replan loop** — build a data pipeline over SQL that computes rolling baselines, flags anomalies before they're obvious, and understand where control towers, exception management, and AI-assisted (including agentic) automation fit in a modern operations function.

## Prerequisites

- Comfortable with `SELECT`, `WHERE`, `GROUP BY`, `HAVING`, joins, `CASE`, and window functions (`AVG() OVER`, `ROW_NUMBER()`) in SQL — Weeks 2–10 of this course, or [C33 Crunch SQL](../../../C33-CRUNCH-SQL/).
- Comfortable with pandas: `read_sql`, `groupby`, rolling windows (`.rolling()`), and basic arithmetic on DataFrame columns.
- Week 5's safety-stock and service-level formulas, Week 6's supplier scorecard concepts, and Week 7's mode/carrier cost figures — this week's disruption simulation reuses all three directly.
- PostgreSQL 16+ **or** SQLite 3.35+, plus Python 3.10+ with `pandas` and `numpy`. See [`resources.md`](./resources.md) for install steps.

## Setup — seed the risk register and the operating-metrics ledger

Everything this week runs against two tables. Create both once, before Lecture 1.

**PostgreSQL:**

```bash
createdb crunch_chain_wk11
psql crunch_chain_wk11
```

**SQLite:**

```bash
sqlite3 crunch_chain_wk11.db
```

### Table 1 — `network_risk_register`

Sixteen named risks across Crunch Gear's network — the register you'll score in Lecture 1 and Exercise 1. Run this (identical on both engines):

```sql
CREATE TABLE network_risk_register (
    risk_id                    INTEGER PRIMARY KEY,
    node_name                  TEXT    NOT NULL,
    node_type                  TEXT    NOT NULL,   -- Facility, Supplier, Carrier, System, Market
    risk_category               TEXT    NOT NULL,   -- Supplier, Facility, Transportation, Cyber, Geopolitical, Demand
    risk_description            TEXT    NOT NULL,
    likelihood                  INTEGER NOT NULL,   -- 1 (rare) - 5 (near-certain within a year)
    impact                      INTEGER NOT NULL,   -- 1 (negligible) - 5 (severe)
    single_point_of_failure     BOOLEAN NOT NULL,
    annual_volume_share_pct     NUMERIC,             -- % of relevant network volume/spend this node carries, where known
    mitigation_status           TEXT    NOT NULL    -- None, Partial, Mitigated
);
```

The full 16-row `INSERT` is in [`resources.md`](./resources.md#full-network_risk_register-seed-data) to keep this page short — here's a representative slice:

```sql
INSERT INTO network_risk_register VALUES
(1, 'Andes Stitch Works', 'Supplier', 'Supplier', 'Sole qualified CMT line for the flagship Outerwear style; no second line trained on the pattern', 3, 4, TRUE, 35, 'Partial'),
(3, 'Hai Phong CM Region', 'Facility', 'Geopolitical', 'Sole offshore contract-manufacturing region and import gateway; typhoon season closes the port 3-5 times/yr', 4, 5, TRUE, 60, 'None'),
(5, 'Austin East DC', 'Facility', 'Facility', 'Largest DC by volume; single ERCOT grid connection, no generator failover, winter-storm exposure', 3, 5, TRUE, 45, 'None'),
(8, 'Order Management System', 'System', 'Cyber', 'Single ERP/OMS instance; no hot failover; ransomware or outage halts order release network-wide', 2, 5, TRUE, 100, 'None');
-- ... 12 more rows — full block in resources.md
```

Sanity check — this should print `16`:

```sql
SELECT COUNT(*) FROM network_risk_register;
```

### Table 2 — `daily_ops`

Sixty days (April 1 – May 30, 2026) of daily operating metrics for all three DCs — **180 rows**. The disruption is real inside this data: watch for it in Lecture 1, measure it precisely in Exercise 2.

```sql
CREATE TABLE daily_ops (
    ops_id              INTEGER PRIMARY KEY,
    ops_date            DATE    NOT NULL,
    dc                  TEXT    NOT NULL,   -- 'Austin East', 'Memphis DC', 'Reno DC'
    orders_received     INTEGER NOT NULL,
    units_shipped       INTEGER NOT NULL,
    otif_pct            NUMERIC NOT NULL,   -- on-time-in-full %, share of orders shipped complete and on time
    fill_rate_pct       NUMERIC NOT NULL,   -- % of ordered units actually shipped that day
    avg_lead_time_days  NUMERIC NOT NULL,   -- order-to-ship, DC average that day
    inbound_delay_flag  BOOLEAN NOT NULL    -- TRUE if an inbound replenishment shipment arrived late that day
);
```

The full 180-row `INSERT` is in [`resources.md`](./resources.md#full-daily_ops-seed-data) — here's the opening slice so you can see the shape:

```sql
INSERT INTO daily_ops VALUES
(1,'2026-04-01','Austin East',156,1445,95.5,96.7,2.0,FALSE),
(2,'2026-04-01','Memphis DC',120,797,94.7,97.0,2.2,FALSE),
(3,'2026-04-01','Reno DC',104,955,92.8,94.6,2.4,FALSE),
(4,'2026-04-02','Austin East',149,942,95.5,96.9,2.2,FALSE),
(5,'2026-04-02','Memphis DC',126,1005,95.2,95.1,2.3,FALSE),
(6,'2026-04-02','Reno DC',108,702,94.6,94.0,2.6,FALSE);
-- ... 174 more rows — full block in resources.md
```

Sanity check — this should print `180`:

```sql
SELECT COUNT(*) FROM daily_ops;
```

**The scenario, precisely:** Andes Stitch Works has a production fire on **2026-04-15** and is down for **18 days** (resumes 2026-05-03). Austin East's finished-Outerwear safety stock absorbs the first ~12 days; visible service damage starts **2026-04-27**, bottoms out **2026-05-04 to 2026-05-10** (OTIF as low as **61.0%**, fill rate as low as **66.1%**, average lead time up to **6.5 days**), and recovers to a sustained 95%+ OTIF by **2026-05-20**. Memphis DC and Reno DC — which don't depend on that CMT line — barely move. That isolation is itself a finding: a single point of failure, by definition, doesn't take down the whole network, just the part that depends on it.

## Weekly schedule

Adds up to roughly the course's **~28 hr/week full-time pace**.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|--------------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Risk taxonomy, SPOFs, the risk matrix | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Tuesday | Control towers, pipelines, replanning cadence | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Wednesday | Disruption simulation; cost + recovery time | 0h | 1.5h | 0h | 0.5h | 1h | 0h | 3h |
| Thursday | AI in ops — anomaly detection, exceptions | 2h | 1h | 1.5h | 0.5h | 1h | 1h | 7h |
| Friday | Resilience redesign; catch-up | 0h | 0h | 1.5h | 0.5h | 1h | 1.5h | 4.5h |
| Saturday | Mini-project — stress test + replan pipeline | 0h | 0h | 0h | 0h | 0h | 2.5h | 2.5h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **4h** | **5h** | **3h** | **3.5h** | **5h** | **5h** | **27.5h** |

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-supply-chain-risk-and-resilience.md](./lecture-notes/01-supply-chain-risk-and-resilience.md) | Risk taxonomy, single points of failure, the likelihood × impact matrix, resilience levers | 2h |
| 2 | [lecture-notes/02-digital-supply-chain-and-control-towers.md](./lecture-notes/02-digital-supply-chain-and-control-towers.md) | Control towers, automated pipelines over SQL, monthly cycles → continuous replanning | 2h |
| 3 | [lecture-notes/03-ai-in-operations.md](./lecture-notes/03-ai-in-operations.md) | Anomaly detection, ML-driven exception management, agentic automation in the replan loop | 2h |
| 4 | [exercises/exercise-01-risk-scoring-model.md](./exercises/exercise-01-risk-scoring-model.md) | Score all 16 network risks, build the matrix, shortlist "must mitigate first" | 1.5h |
| 5 | [exercises/exercise-02-disruption-simulation.md](./exercises/exercise-02-disruption-simulation.md) | Measure the Andes Stitch Works fire's cost and recovery time from `daily_ops` | 1.5h |
| 6 | [exercises/exercise-03-automate-a-daily-ops-pipeline.md](./exercises/exercise-03-automate-a-daily-ops-pipeline.md) | Build a rolling-baseline anomaly-flagging pipeline in SQL + Python | 1h |
| 7 | [challenges/challenge-01-resilience-network-redesign.md](./challenges/challenge-01-resilience-network-redesign.md) | Design and cost a dual-sourcing + buffer resilience plan for the top risk cluster | 1.5h |
| 8 | [challenges/challenge-02-anomaly-detection-on-ops.md](./challenges/challenge-02-anomaly-detection-on-ops.md) | Tune a real anomaly detector across all 3 DCs; measure detection lag and false positives | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Stress-test a disruption + build an automated replan pipeline with alerting | 2.5h |
| 10 | [homework.md](./homework.md) | Extra practice tying risk, disruption, and automation together | 5h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Full seed data, official references, tools to install | — |

## By the end of this week you can…

- Build a risk register for a real network and explain why "high likelihood, high impact, single point of failure" is a fundamentally different priority than "high impact, low likelihood, well-diversified."
- Take a named disruption and a real operating-data table and produce a defensible dollar cost and a precise recovery-time figure — not "it was bad for a while."
- Propose a resilience plan (dual-source, buffer, redundancy) and show, with an expected-annual-loss calculation, whether its cost is justified by the risk it removes.
- Write a SQL query that computes a rolling baseline and flags a day as anomalous relative to it — the same mechanism that powers a real control-tower alert feed.
- Explain, in concrete terms, where anomaly detection, exception management, and agentic AI actually fit in the plan-forecast-replan loop — and where a human still has to be in it.

## Up next

Week 12 — Capstone: profile and optimize a full network end to end, cutting total landed cost against a service constraint. Every lever from Weeks 1–11 — forecasting, inventory, procurement, logistics, warehousing, optimization, S&OP, and this week's risk and automation — comes back into one project.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
