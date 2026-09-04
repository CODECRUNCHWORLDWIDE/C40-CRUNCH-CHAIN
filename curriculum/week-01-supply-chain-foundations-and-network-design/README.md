# Week 1 — Supply Chain Foundations & Network Design

> **Goal:** by Sunday you can look at any product — a jacket, a phone, a bag of coffee — and draw the network that got it to a customer: every node, every echelon, every flow of product, information, and cash. You can name the four KPIs that run an operations team's dashboard, compute each one by hand, and argue centralized vs. distributed network design with real trade-offs instead of a gut feeling.

Welcome to **C40 · Crunch Chain**. Every later week in this course — forecasting, inventory, procurement, logistics, warehousing, optimization, S&OP — is a deeper answer to a question this week asks first: *what does this network look like, and how do we know if it's working?* Skim this week and everything after it feels like memorized formulas with no home. Learn it properly and every later formula has a place to live.

We use one running example all week: **Crunch Gear**, a fictional outdoor-apparel company that buys fabric from mills, has jackets sewn by contract manufacturers, ships them through regional warehouses, and sells to both wholesale retailers and its own direct-to-consumer (DTC) site. It is small enough to hold in your head and realistic enough that every technique here transfers directly to a real job.

**Data rule for this course:** whenever we store, manage, or query records — orders, shipments, inventory, financials — we use **SQL (PostgreSQL 16, SQLite fallback)** and/or **Python (pandas)**. We never use a spreadsheet as a data store. Week 1's exercises are mostly pencil-and-paper (you're learning to reason before you learn to query), but the mini-project's dataset already ships as a SQL seed script, because that's how you'll receive real operations data starting Week 2.

## Learning objectives

By the end of this week, you will be able to:

- **Map** an end-to-end supply chain — suppliers, plants, warehouses, lanes, and customers — and identify its echelons and the three flows (product, information, cash) moving through it.
- **Define and compute** the core operational KPIs — OTIF, fill rate, perfect order, inventory turns, and cash-to-cash cycle time — and state what each one reveals about the business.
- **Reason** about network shape: how many echelons a chain needs, centralized vs. distributed facility strategy, and the fundamental cost-service trade-off that shapes every network decision.
- **Explain** the bullwhip effect in plain terms — why a small wiggle in consumer demand becomes a large swing in factory orders — and why it matters before you've even priced a lane or run a forecast.
- **Frame** every topic ahead in this course as a decision that should be driven by data pulled from a real table, not a hunch or a spreadsheet guess.

## Standards this week meets

| Bar | What this week is measured against |
| --- | --- |
| University | `MAN 3025` — describe a supply chain end to end, its echelons and its three flows, and define and compute the operating measures by which the chain is judged. |
| Industry | Produce the weekly operating numbers a manager asks for — on-time-in-full, fill rate, perfect order, turns, cash-to-cash — from a raw order table, and say which one is lying and why. |
| Beyond the bar | The five KPIs are computed by hand before a single query is written, so that when the same numbers come back out of a database in Week 02 the learner can sanity-check them on sight — `exercises/exercise-02-compute-core-kpis-by-hand.md` |

## Prerequisites

- You can run commands in a terminal and read basic Python (per the course syllabus) — you won't need either heavily this week, but the mini-project introduces the SQL seed pattern you'll use from Week 2 onward.
- No prior supply-chain background assumed. This week starts at zero.
- A way to run SQL against the mini-project's seed data helps but isn't required to *start* — PostgreSQL 16+ or SQLite 3.35+. See [`resources.md`](./resources.md) for install steps. If you have neither installed yet, you can still do the hand-computation exercises and come back for the mini-project.

## Setup

Nothing to install to begin. Lectures 1–3 and Exercises 1–3 are worked with pencil, paper, and a calculator (or a spreadsheet-free back-of-envelope — see why we say "back of envelope" and not "spreadsheet" in Lecture 2). The mini-project supplies a small SQL seed script; if you want to get PostgreSQL or SQLite ready now, jump to [`resources.md`](./resources.md).

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-what-is-a-supply-chain.md](./lecture-notes/01-what-is-a-supply-chain.md) | Nodes, echelons, the three flows, where cost and value accumulate, a first look at the bullwhip effect | 2h |
| 2 | [lecture-notes/02-supply-chain-metrics-and-kpis.md](./lecture-notes/02-supply-chain-metrics-and-kpis.md) | OTIF, fill rate, perfect order, inventory turns, cash-to-cash cycle — formulas, worked examples, the lever each one points to | 2h |
| 3 | [lecture-notes/03-network-design-and-the-cost-service-tradeoff.md](./lecture-notes/03-network-design-and-the-cost-service-tradeoff.md) | Echelon count, centralized vs. distributed, the square root law, cross-docking, the cost-service curve | 2h |
| 4 | [exercises/exercise-01-map-a-real-chain.md](./exercises/exercise-01-map-a-real-chain.md) | Map a real product's chain end to end, nodes and flows | 1h |
| 5 | [exercises/exercise-02-compute-core-kpis-by-hand.md](./exercises/exercise-02-compute-core-kpis-by-hand.md) | Compute OTIF, fill rate, perfect order, turns, and C2C from a 10-order table | 1.5h |
| 6 | [exercises/exercise-03-sketch-two-network-designs.md](./exercises/exercise-03-sketch-two-network-designs.md) | Design and compare a centralized vs. distributed network for a growing DTC brand | 1h |
| 7 | [challenges/challenge-01-diagnose-a-broken-kpi.md](./challenges/challenge-01-diagnose-a-broken-kpi.md) | A dashboard looks fine except one number — find the real problem | 1h |
| 8 | [challenges/challenge-02-argue-centralize-vs-distribute.md](./challenges/challenge-02-argue-centralize-vs-distribute.md) | Write and defend a network recommendation with numbers | 1h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Map a real chain end to end + compute KPIs from a supplied SQL dataset | 3h |
| 10 | [homework.md](./homework.md) | Extra practice, spaced across the week | 4.5h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official/free references + tools to install | — |

## Weekly schedule

Adds up to roughly the course's **~28 hr/week full-time pace**. Adjust to your own pace per the syllabus.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|-------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | What a supply chain is; nodes and flows | 2h | 1h | 0h | 0.5h | 1h | 0h | 4.5h |
| Tuesday | KPIs — OTIF, fill rate, perfect order | 2h | 1.5h | 0h | 0.5h | 1h | 0h | 5h |
| Wednesday | KPIs — inventory turns, cash-to-cash | 0h | 0h | 1h | 0.5h | 1h | 1h | 3.5h |
| Thursday | Network design & cost-service trade-off | 2h | 1h | 1h | 0.5h | 1h | 1h | 6.5h |
| Friday | Sketch networks; challenges | 0h | 1h | 1h | 0.5h | 1h | 1h | 4.5h |
| Saturday | Mini-project | 0h | 0h | 0h | 0h | 0h | 3h | 3h |
| Sunday | Quiz + review | 0h | 0h | 0h | 1h | 0h | 0h | 1h |
| **Total** | | **6h** | **4.5h** | **3h** | **3.5h** | **5h** | **6h** | **28.5h** |

## By the end of this week you can…

- Draw a real supply chain's nodes, echelons, and three flows without hesitating on what goes where.
- Compute OTIF, fill rate, perfect order, inventory turns, and cash-to-cash cycle time from a raw order table and explain what each number means to a non-technical manager.
- Argue, with numbers, whether a growing brand should centralize or distribute its warehousing — and know that "it depends" has a specific, calculable answer.
- Recognize the bullwhip effect when you see order volatility that doesn't match sell-through.

## Up next

[Week 2 — SQL & pandas data foundations for operations](../week-02-sql-and-pandas-data-foundations/) — everything you reasoned about on paper this week, you'll model as real tables and query for real.

---

*Part of the Code Crunch Worldwide open curriculum · GPL-3.0 · If you find errors, please open an issue or PR.*
