# Week 4 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific question comes up.

## Install first

This week is Python-first. Set up a virtual environment once:

```bash
python3 -m venv .venv && source .venv/bin/activate
pip install pandas numpy statsmodels scikit-learn psycopg2-binary sqlalchemy matplotlib
```

- **PostgreSQL 16+** (if not already installed from earlier weeks) — <https://www.postgresql.org/download/> · macOS: [Postgres.app](https://postgresapp.com/).
- **SQLite 3.35+** — zero-setup fallback, ships on macOS/most Linux: <https://www.sqlite.org/download.html>.
- **`statsmodels`** — Holt-Winters and OLS regression this week: <https://www.statsmodels.org/stable/index.html>
- **`scikit-learn`** — gradient-boosted trees and metrics: <https://scikit-learn.org/stable/>

## Required reading (this week's core)

- **`statsmodels` — Exponential Smoothing (Holt-Winters):**
  <https://www.statsmodels.org/stable/examples/notebooks/generated/exponential_smoothing.html>
  *Why: the official worked example, covers additive/multiplicative and damped trend directly.*
- **`statsmodels` — `ExponentialSmoothing` API reference:**
  <https://www.statsmodels.org/stable/generated/statsmodels.tsa.holtwinters.ExponentialSmoothing.html>
  *Why: every parameter this week's code uses (`trend`, `damped_trend`, `seasonal`, `seasonal_periods`, `initialization_method`), explained precisely.*
- **`statsmodels` — Variance Inflation Factor:**
  <https://www.statsmodels.org/stable/generated/statsmodels.stats.outliers_influence.variance_inflation_factor.html>
  *Why: the exact collinearity diagnostic this week's regression lecture uses.*
- **scikit-learn — `TimeSeriesSplit`:**
  <https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.TimeSeriesSplit.html>
  *Why: `scikit-learn`'s own built-in rolling-origin splitter — a library-provided alternative to the hand-written version this week has you build first (build it by hand once so you understand exactly what it's doing, then feel free to reach for this in real work).*
- **scikit-learn — `GradientBoostingRegressor`:**
  <https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.GradientBoostingRegressor.html>
  *Why: full parameter reference — `n_estimators`, `max_depth`, `learning_rate`, and the loss functions available.*

## Core concepts, explained well elsewhere

- **Hyndman & Athanasopoulos, *Forecasting: Principles and Practice* (free online textbook)** — the standard reference for this entire week:
  - Exponential smoothing (incl. Holt-Winters): <https://otexts.com/fpp3/expsmooth.html>
  - Time series regression models: <https://otexts.com/fpp3/regression.html>
  - Time series cross-validation: <https://otexts.com/fpp3/tscv.html>
  - Intermittent demand (Croston and variants): <https://otexts.com/fpp3/counts.html>
  - Forecast reconciliation (bottom-up, top-down, MinT): <https://otexts.com/fpp3/hierarchical.html>
  *Why: this is the single best free resource for everything this week covers, written by two of the field's leading researchers. If you only read one outside link this week, read the reconciliation chapter before Challenge 2.*
- **"Lookahead bias" and backtesting pitfalls (general, finance-oriented but directly applicable):**
  <https://en.wikipedia.org/wiki/Look-ahead_bias>
  *Why: the concept behind Lecture 3's `.shift(1)` warning, explained from the angle where it was first named.*
- **Syntetos & Boylan, on the Croston bias correction (SBA)** — search "Syntetos-Boylan Approximation" for accessible summaries; the original paper is paywalled, but most demand-planning textbooks and several open lecture notes summarize the correction clearly.
  *Why: Challenge 1 asks you to name and describe this — this is where to look it up.*
- **Rob Hyndman's blog, "Forecast reconciliation" tag:**
  <https://robjhyndman.com/hyndsight/reconciliation/>
  *Why: shorter, blog-length treatments of MinT and hierarchical forecasting if the textbook chapter above is more than you need.*

## Reference (keep in tabs)

- **`pandas` — Time series / date functionality:** <https://pandas.pydata.org/docs/user_guide/timeseries.html>
  *Why: `asfreq`, `shift`, `rolling`, and the date-offset aliases (`W-MON`, etc.) used throughout this week's code.*
- **`statsmodels` — OLS regression results guide:** <https://www.statsmodels.org/stable/regression.html>
  *Why: how to read the full `summary()` output — R², coefficient table, confidence intervals.*
- **scikit-learn — Model evaluation: `mean_absolute_error` and friends:**
  <https://scikit-learn.org/stable/modules/model_evaluation.html#regression-metrics>
  *Why: MAE, MAPE-adjacent metrics, and where scikit-learn's built-ins fall short for intermittent demand (worth knowing what's *not* built in).*

## Practice beyond this week's dataset

- **Kaggle — M5 Forecasting Accuracy competition (data + writeups):**
  <https://www.kaggle.com/competitions/m5-forecasting-accuracy>
  *Why: the real-world competition that popularized rolling-origin backtesting and gradient-boosted trees for retail demand forecasting at scale — read a few top writeups for how professionals structure exactly this week's workflow on messier, larger data.*
- **`statsmodels` example notebooks (browse the full gallery):**
  <https://www.statsmodels.org/stable/examples/index.html#time-series-analysis>
  *Why: more worked examples beyond Holt-Winters if you want extra reps before the mini-project.*

## Deeper background (optional this week)

- **Holt (1957) and Winters (1960), the original papers** — hard to find free full text, but most forecasting textbooks (including the Hyndman/Athanasopoulos text above) summarize the original derivation faithfully in the exponential smoothing chapter.
  *Why: understand *why* the equations look the way they do, not just how to call `.fit()`.*
- **Friedman (2001), "Greedy Function Approximation: A Gradient Boosting Machine"** — the original gradient boosting paper:
  <https://projecteuclid.org/euclid.aos/1013203451>
  *Why: dense but foundational if you want to go beyond "call `GradientBoostingRegressor`" to understanding what the algorithm is actually doing at each iteration.*
