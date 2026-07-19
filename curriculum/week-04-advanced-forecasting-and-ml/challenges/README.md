# Week 4 Challenges — Overview

Two challenges, each a hard case none of this week's three main lectures handle out of the box.

| # | Challenge | Problem | Time | Difficulty |
|--:|-----------|---------|-----:|------------|
| 1 | [challenge-01-intermittent-demand-croston.md](./challenge-01-intermittent-demand-croston.md) | A SKU that sells on ~1 week in 3, everything else is zero | 90 min | Medium |
| 2 | [challenge-02-hierarchical-forecast-reconciliation.md](./challenge-02-hierarchical-forecast-reconciliation.md) | SKU-level and total-level forecasts that must add up | 90 min | Hard |

## Why these two, specifically

Every model in this week's lectures — Holt-Winters, regression, gradient-boosted trees — silently assumes the series is "mostly continuous": there's a meaningful level, trend, and season to estimate every period. Two extremely common real situations break that assumption outright:

- **Intermittent demand** (Challenge 1): most periods have *zero* demand, punctuated by occasional nonzero spikes. A moving average or Holt-Winters fit on a mostly-zero series produces a forecast that's technically a number but operationally useless — it doesn't answer the question a planner actually has, which is "how often does this thing sell, and how much when it does?"
- **Hierarchical structure** (Challenge 2): you don't have one series, you have many that are related by addition — SKU-level series that sum to a total, or region-level series that sum to a company total. Forecast each independently and the numbers **will not add up** — a director looking at "sum of the SKU forecasts" and "the total forecast" side by side and seeing two different numbers is a fast way to lose credibility with the exact people who need to trust your model.

Both challenges use the same `weekly_demand` table as the rest of the week — no new data to load.

## How to work these

Same discipline as prior weeks' challenges: state your reasoning, don't just paste code. Both challenges have real, defensible tradeoffs rather than one clean right answer — the grading rubric in each file rewards showing your work and being honest about a method's weaknesses, not just getting a number.

## Submission

Commit each challenge's deliverable to your portfolio under `c40-week-04/challenge-0N/`, per that challenge's own instructions.
