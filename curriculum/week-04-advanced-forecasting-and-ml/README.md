# Week 4 — Advanced Forecasting & ML

> **Goal:** by Sunday you can take a demand series that a naive or moving-average model handles badly — trended and seasonal, or driven by price and promotions, or lumpy and intermittent, or split across a product/location hierarchy — and forecast it properly: Holt-Winters for trend+season, regression for causal drivers, a tuned tree ensemble backtested on a rolling origin, Croston for intermittent parts, and top-down/bottom-up reconciliation across a hierarchy. You can also tell me, with a number, whether all that machinery actually beat the boring baseline from Week 3.

Welcome back to **C40 · Crunch Chain**. Week 3 gave you the forecasting toolbox everyone should reach for first: naive, moving average, seasonal naive, simple exponential smoothing, and the discipline of scoring a forecast against a holdout with MAPE and bias instead of eyeballing a chart. Those models are fast, transparent, and — this is the part people forget — **often correct**. This week is not about replacing them. It's about knowing when they're not enough, and having the next tool ready when that's true: a trended-and-seasonal series that a flat moving average chases badly, a series where price and promotions move demand more than the calendar does, a series with enough structure that a tree ensemble can learn nonlinear interactions a straight line can't, a part that sells three units some weeks and zero most weeks, and a catalog where SKU-level and total-level forecasts have to agree with each other.

We keep using the same **Crunch Gear** demand data from Weeks 2–3: weekly units sold by SKU and region, alongside price and promotion flags. This week adds three SKUs with genuinely different shapes — a trended, winter-peaking jacket; a summer-peaking tent that responds hard to promotions; and a repair-parts kit that sells so rarely most weeks show zero — because different demand shapes need different models, and pretending one method fits all of them is how forecasting teams lose credibility with the planners who depend on their numbers.

**Data rule for this course:** every table lives in **SQL (PostgreSQL 16, SQLite fallback)**, every model is fit in **Python (pandas + statsmodels + scikit-learn)**. We never use a spreadsheet as a data store or a forecasting engine — Excel's linear trendline and "seasonal adjustment" checkbox hide exactly the assumptions this week teaches you to state explicitly and test.

## Learning objectives

By the end of this week, you will be able to:

- **Apply** Holt-Winters triple exponential smoothing to a trended, seasonal demand series — fit it in `statsmodels`, read off level/trend/seasonal smoothing parameters, and diagnose whether the fit is capturing real structure or overfitting noise.
- **Build** a causal, feature-based forecast using price, promotion, and calendar features with linear regression — and recognize when the resulting coefficients are confounded rather than causal (a trap real forecasting teams fall into constantly).
- **Train and tune** a tree-ensemble forecaster (gradient boosting) on a lag/rolling/calendar feature matrix, and **backtest it correctly** with rolling-origin cross-validation so the reported accuracy isn't inflated by leakage.
- **Handle two hard cases**: intermittent demand with Croston's method (when most periods are zero) and hierarchical reconciliation (when SKU-level and total-level forecasts must add up).
- **Choose the winning model with evidence** — compare MAE/MAPE across models on the same holdout, and know when Week 3's simple baseline is still the right call because the fancier model didn't actually win.

## Prerequisites

- Week 3 complete: you can compute and interpret MAPE and bias, and you've fit at least a moving-average and simple-exponential-smoothing baseline against a holdout.
- Comfort with the `weekly_demand`-style table from Weeks 2–3 (or the reseed script below if you're starting fresh here).
- Python with `pandas`, `statsmodels`, and `scikit-learn` installed (see [`resources.md`](./resources.md)). Everything this week is Python-first; SQL is for storing and pulling the data, not for fitting models.

## Setup

Install the Python stack once for the week:

```bash
python3 -m venv .venv && source .venv/bin/activate
pip install pandas numpy statsmodels scikit-learn psycopg2-binary sqlalchemy matplotlib
```

If you don't already have this week's demand data loaded, the [mini-project](./mini-project/README.md) includes a full, reproducible generator (`generate_demand.py`, seeded, deterministic) that builds the `weekly_demand` table used across every lecture, exercise, and challenge this week. Run it once at the start of the week — every file below assumes it exists.

```sql
-- sanity check after running the generator
SELECT sku, region, COUNT(*) AS weeks, MIN(week_start), MAX(week_start)
FROM weekly_demand
GROUP BY sku, region
ORDER BY sku, region;
-- expect 3 SKUs x 2 regions x 156 weeks = 936 rows total
```

## How to navigate this week

Work top to bottom. Each piece assumes the ones above it — the regression lecture assumes you've seen the Holt-Winters diagnostics, and the backtesting lecture assumes both.

| # | File | What's inside | ~Time |
|--:|------|---------------|------:|
| 1 | [lecture-notes/01-holt-winters-and-seasonality.md](./lecture-notes/01-holt-winters-and-seasonality.md) | Triple exponential smoothing — level, trend, seasonal components; fitting and diagnosing it in `statsmodels` | 2.5h |
| 2 | [lecture-notes/02-causal-and-feature-based-forecasting.md](./lecture-notes/02-causal-and-feature-based-forecasting.md) | Regression with price, promo, calendar features; turning demand history into a supervised-learning problem; confounding | 2.5h |
| 3 | [lecture-notes/03-ml-forecasting-and-backtesting.md](./lecture-notes/03-ml-forecasting-and-backtesting.md) | Tree-ensemble forecasters, rolling-origin cross-validation, leakage | 2.5h |
| 4 | [exercises/exercise-01-fit-holt-winters.md](./exercises/exercise-01-fit-holt-winters.md) | Fit Holt-Winters on a real SKU, read the params, forecast 13 weeks | 1.5h |
| 5 | [exercises/exercise-02-regression-with-promo-features.md](./exercises/exercise-02-regression-with-promo-features.md) | Build the feature matrix, fit OLS, interpret (and question) the coefficients | 1.5h |
| 6 | [exercises/exercise-03-rolling-origin-backtest.md](./exercises/exercise-03-rolling-origin-backtest.md) | Implement a rolling-origin backtest loop from scratch, no shortcuts | 2h |
| 7 | [challenges/challenge-01-intermittent-demand-croston.md](./challenges/challenge-01-intermittent-demand-croston.md) | Forecast a spare-parts SKU with Croston's method | 1.5h |
| 8 | [challenges/challenge-02-hierarchical-forecast-reconciliation.md](./challenges/challenge-02-hierarchical-forecast-reconciliation.md) | Reconcile SKU x region forecasts up to totals so the numbers add up | 1.5h |
| 9 | [mini-project/README.md](./mini-project/README.md) | Backtested ML forecaster vs. the Week 3 baseline, with the seed data generator | 4h |
| 10 | [homework.md](./homework.md) | Extra practice, spaced across the week | 4h |
| 11 | [quiz.md](./quiz.md) | 15 self-check questions + answer key | 1h |
| 12 | [resources.md](./resources.md) | Official/free references + tools to install | — |

## Weekly schedule

Adds up to roughly the course's **~28 hr/week full-time pace**. Adjust to your own pace per the syllabus.

| Day | Focus | Lectures | Exercises | Challenges | Quiz/Read | Homework | Mini-Project | Daily Total |
|-----------|-------------------------------------------|---------:|----------:|-----------:|----------:|---------:|-------------:|------------:|
| Monday | Holt-Winters — level, trend, season | 2.5h | 1.5h | 0h | 0.5h | 1h | 0h | 5.5h |
| Tuesday | Causal regression; confounding | 2.5h | 1.5h | 0h | 0.5h | 1h | 0h | 5.5h |
| Wednesday | Tree ensembles; rolling-origin backtest | 2.5h | 2h | 0h | 0.5h | 1h | 0h | 6h |
| Thursday | Intermittent demand; hierarchical reconciliation | 0h | 0h | 3h | 0.5h | 1h | 1h | 5.5h |
| Friday | Mini-project: build + backtest + prove it | 0h | 0h | 0h | 0.5h | 1h | 3h | 4.5h |

## What you already know vs. what's new this week

| Week 3 gave you | Week 4 adds |
|---|---|
| Naive, moving average, seasonal naive | Holt-Winters triple exponential smoothing (fits trend *and* season together, with smoothing weights) |
| Simple exponential smoothing (level only) | Regression on **causal drivers** — price, promo, calendar — not just past values of the series itself |
| Eyeball + MAPE/bias on one holdout | **Rolling-origin backtesting** — multiple holdouts sliding forward in time, so one lucky/unlucky split can't fool you |
| One model shape fits every SKU | Different tools for different demand shapes: smooth-and-seasonal (Holt-Winters), driver-heavy (regression), complex-nonlinear (ML), sparse (Croston) |
| SKU-level forecasts in isolation | **Hierarchical reconciliation** — SKU and total forecasts that are numerically consistent with each other |

When done: push, take the [quiz](./quiz.md), and move on to [Week 5 — Inventory: EOQ, reorder point, safety stock, service level](../week-05-inventory-eoq-reorder-point-and-safety-stock/).
