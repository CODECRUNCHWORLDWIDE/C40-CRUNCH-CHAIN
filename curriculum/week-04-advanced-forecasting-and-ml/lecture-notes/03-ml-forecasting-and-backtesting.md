# ML Forecasting & Backtesting

Regression (Lecture 2) assumes the relationship between features and demand is a straight line — price has *a* slope, promo has *a* fixed lift, and those effects add together. Real demand often doesn't cooperate: maybe a promo lifts volume more when it lands in peak season than off-season (an *interaction* a linear model can't represent without you hand-building the interaction term), or the relationship between recent sales momentum and next week's demand isn't linear at all. A **tree-ensemble** forecaster — gradient-boosted trees, the same family of model that wins most structured-data ML competitions — learns these interactions and nonlinearities automatically, from a feature matrix built the same way as Lecture 2's, plus one new ingredient: **lag and rolling features**, so the model can use the series' own recent history as inputs, not just external drivers.

This lecture is really two lessons in one, and the second is more important than the first: **how to build the ML feature matrix**, and **how to backtest it without lying to yourself about its accuracy**. Skip the second part and every number you report is worthless — a leaked backtest routinely reports 3–5x better accuracy than the model will ever achieve live, and a team that ships on a leaked number gets burned in production.

## 1. Turning a series into ML features: lags and rolling stats

```python
import pandas as pd, numpy as np

df = pd.read_sql(
    "SELECT week_start, units, unit_price, promo_flag "
    "FROM weekly_demand WHERE sku = 'JCK-100' AND region = 'Northeast' "
    "ORDER BY week_start",
    con=engine,
)
d = df.copy()

for lag in [1, 2, 3, 4, 52]:
    d[f"lag_{lag}"] = d["units"].shift(lag)

d["roll_mean_4"] = d["units"].shift(1).rolling(4).mean()
d["roll_mean_8"] = d["units"].shift(1).rolling(8).mean()

d["weekofyear"] = pd.to_datetime(d["week_start"]).dt.isocalendar().week.astype(int)
d["sin52"] = np.sin(2 * np.pi * d["weekofyear"] / 52)
d["cos52"] = np.cos(2 * np.pi * d["weekofyear"] / 52)
d["promo"] = d["promo_flag"].astype(int)
d["log_price"] = np.log(d["unit_price"])
```

**Read the `.shift(1)` before `.rolling(4)` twice — it's the single most common leakage bug in ML forecasting.** `d["units"].rolling(4).mean()` *without* the shift computes each row's 4-week average **including that row's own actual value** — meaning the "feature" for week 40 partly consists of week 40's own answer. A model trained on that feature will look spectacular in training and fall apart the moment it has to forecast a week whose actual value isn't known yet (which is every real forecast). `.shift(1)` first pushes the window back one week, so `roll_mean_4` at week 40 only ever uses weeks 36–39 — information genuinely available *before* week 40 happens. This exact bug (forgetting the shift) is called **lookahead bias**, and it's worth building a habit: any time you compute a rolling statistic for use as a forecasting feature, ask "does this window include the row I'm trying to predict?" before you trust it.

`lag_1`...`lag_4`, `lag_52` are also leakage-safe by construction — a lag is inherently "value from N periods ago," so there's no way to accidentally include the current row. Drop the leading rows that don't have a full lag history yet (`dropna`), same as any lag-feature workflow.

## 2. Fit a gradient-boosted tree ensemble

```python
from sklearn.ensemble import GradientBoostingRegressor

feat_cols = ["lag_1", "lag_2", "lag_3", "lag_4", "lag_52",
             "roll_mean_4", "roll_mean_8", "sin52", "cos52", "promo", "log_price"]
d2 = d.dropna(subset=feat_cols).reset_index(drop=True)

model = GradientBoostingRegressor(
    n_estimators=200, max_depth=3, learning_rate=0.05, random_state=40,
)
model.fit(d2.loc[:90, feat_cols], d2.loc[:90, "units"])
```

`n_estimators` (how many trees), `max_depth` (how deep each tree, i.e. how many interactions it can represent), and `learning_rate` (how much each tree corrects the previous ensemble's error) are the three knobs that matter most. Shallow trees (`max_depth=2–4`) plus a low `learning_rate` (0.02–0.1) plus enough `n_estimators` to compensate is the standard, robust starting recipe — it resists overfitting far better than a few deep, aggressive trees, because each individual tree is only allowed to make a small correction. Tune these with the backtest below, never with a single train/test split.

**Read the feature importances — they tell you what the model actually learned to rely on:**

```python
imp = sorted(zip(feat_cols, model.feature_importances_), key=lambda x: -x[1])
for f, v in imp:
    print(f"{f:>12}: {v:.3f}")
```

```
       sin52: 0.515
       lag_1: 0.210
 roll_mean_4: 0.104
       lag_2: 0.058
       lag_4: 0.022
   log_price: 0.021
       promo: 0.020
       cos52: 0.020
 roll_mean_8: 0.014
      lag_52: 0.013
       lag_3: 0.005
```

Two-thirds of the model's decisions trace back to `sin52` and `lag_1` alone — the calendar position and last week's value. That's a useful sanity check, not just a curiosity: if a feature you expected to matter (say, `promo`) shows near-zero importance, either the model genuinely found it uninformative once other features were present (plausible — `sin52` might already explain most of what `promo` would), or there's a bug in how that feature was built. Always look at this table before trusting a model; a feature importance ranking that makes no domain sense is often the first sign of a leakage or encoding bug, faster to catch here than by staring at accuracy numbers alone.

## 3. Why ordinary k-fold cross-validation is wrong for time series

`scikit-learn`'s default cross-validation, `KFold(shuffle=True)`, randomly splits rows into folds and trains on some, tests on others — the standard approach for i.i.d. data (customer records, product listings, anything where row order doesn't matter). **Never use shuffled k-fold on a time series.** Two reasons, both fatal:

1. **Direct leakage through lag/rolling features.** If week 80 lands in the training fold and week 79 lands in the test fold, the model trains on a row (`lag_1` at week 80) that *contains* week 79's actual value as a feature — while week 79 is simultaneously a row you're pretending to forecast. The model gets to see the answer to the question you're asking it.
2. **Leakage through autocorrelation, even without lag features.** Demand this week and demand last week are correlated even setting lag features aside (they share the same underlying trend and season). A model trained on a random half of all weeks and tested on the other half is being tested on rows that are statistically similar to rows it already trained on — nothing like the real task of forecasting weeks that haven't happened yet.

Either way, shuffled cross-validation reports an accuracy number that has nothing to do with how the model will perform live, and it is almost always **too optimistic** — sometimes dramatically so.

## 4. The right tool: rolling-origin backtesting

**Rolling-origin cross-validation** (also called *walk-forward validation*): pick a series of cutoff points (the "origins") marching forward through time. At each origin, train only on data **before** it, forecast a fixed horizon **after** it, score that forecast, then slide the origin forward and repeat. Every fold respects the one rule that matters: **the model never sees anything from the future relative to its own forecast origin.**

```python
from sklearn.metrics import mean_absolute_error

n = len(d2)
fold_size = 13          # 13-week horizon, matches this course's holdout convention
n_folds = 4
maes_gbm, maes_snaive = [], []

for k in range(n_folds):
    test_end = n - k * fold_size
    test_start = test_end - fold_size
    train_end = test_start
    if train_end < 60:        # stop once there isn't enough history to train on
        break

    Xtr, ytr = d2.loc[:train_end - 1, feat_cols], d2.loc[:train_end - 1, "units"]
    Xte, yte = d2.loc[test_start:test_end - 1, feat_cols], d2.loc[test_start:test_end - 1, "units"]

    m = GradientBoostingRegressor(n_estimators=200, max_depth=3, learning_rate=0.05, random_state=40)
    m.fit(Xtr, ytr)
    pred = m.predict(Xte)
    maes_gbm.append(mean_absolute_error(yte, pred))

    snaive_pred = d2.loc[test_start:test_end - 1, "lag_52"]   # seasonal-naive baseline, same folds
    maes_snaive.append(mean_absolute_error(yte, snaive_pred))

print("GBM mean MAE:   ", round(np.mean(maes_gbm), 1))
print("SNaive mean MAE:", round(np.mean(maes_snaive), 1))
```

Real output on `JCK-100`/Northeast, 3 folds (a 4th didn't have enough training history and was correctly skipped rather than forced):

| Fold | Train size | Test size | GBM MAE | Seasonal-naive MAE |
|---|---:|---:|---:|---:|
| 0 (earliest) | 91 | 13 | 22.9 | 32.5 |
| 1 | 78 | 13 | 14.5 | 27.7 |
| 2 (latest) | 65 | 13 | 16.9 | 27.3 |
| **Mean** | | | **18.1** | **29.2** |

The gradient-boosted model beats the seasonal-naive baseline in every fold, not just on average — that consistency across folds is exactly what a single train/test split can't tell you. One split could have gotten lucky; three independent folds agreeing is real evidence.

## 5. A limitation no amount of tuning fixes: trees can't extrapolate

Look closer at fold 2, the most recent (and highest-volume, since this SKU trends upward): training data tops out at 317 units; the actual test values reach 339. The model's predictions for that fold top out around 306 — **below both the training max and the actual peak.** This is not a tuning problem or a bug — it's structural. A decision tree predicts by averaging the training target values that fall into a leaf; it can never output a number outside the range of values it saw during training, no matter how many trees you add or how you tune them. A linear regression's trend term can extrapolate past its training range (multiply `t` by its coefficient, get a number as large as you like — for better or worse); a tree ensemble fundamentally cannot.

**Consequence for practice:** on a series with a real, ongoing upward trend, a tree ensemble will systematically underforecast the newest, highest periods — exactly the fold-2 pattern above (mean bias ≈ −5.8 units, all in the direction of underforecasting). Two mitigations, both standard: (1) include a detrended target — model `units / rolling_mean` or `units - linear_trend` instead of raw `units`, so the tree only has to learn the *deviation* from a trend that a simple linear term already captures, or (2) blend the tree ensemble with a model that extrapolates properly (Holt-Winters or the regression from Lecture 2) using a weighted average, which the mini-project's "choose the winner" step will ask you to consider explicitly.

## 6. Reading a backtest table honestly

Once you have a table like the one above, three questions before declaring a winner:

- **Is the gap consistent across folds, or driven by one lucky fold?** A model that wins 3 folds by a little is more trustworthy than one that wins 2 folds by a lot and loses 1 badly — the second pattern often means the model is high-variance and got a favorable draw.
- **Is the gap big enough to matter operationally, not just numerically?** An 18.1-vs-29.2 MAE gap on ~190-unit average weekly demand is a real, actionable difference (roughly 10% of average volume). A 0.3-unit MAE gap on the same series would be statistically real but operationally irrelevant — not worth the extra model complexity, monitoring burden, and retraining cadence a tree ensemble requires versus a seasonal-naive baseline anyone on the team can compute by hand.
- **Does the winning model's *type* of error matter more than its *average* error?** A model with lower average MAE that's badly biased low right when the business needs stock the most (peak season, see §5) can be a worse operational choice than a slightly-higher-MAE model with no systematic direction to its error. Check bias (mean signed error), not just MAE, before declaring a winner — this is exactly what the mini-project asks you to do.

Next: put all four models — seasonal naive, Holt-Winters, regression, tree ensemble — on the same rolling-origin backtest and let the evidence, not a hunch, decide the winner. Start with [Exercise 1 — Fit a Holt-Winters Model](../exercises/exercise-01-fit-holt-winters.md).
