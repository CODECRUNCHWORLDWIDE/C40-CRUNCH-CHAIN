# Week 4 — Homework

Five problems, ~4 hours total, spread across the week. A mix of vocabulary, hand-computation, coefficient interpretation, backtest-design critique, and one small research task. Commit each.

---

## Problem 1 — Twenty warm-up questions (45 min)

Short answers, one or two sentences each, in `warmups.md`.

1. Name Holt-Winters' three smoothed components and what each one tracks.
2. What does it mean if a fitted Holt-Winters model has `gamma` very close to 0? Is that always a sign of no seasonality?
3. Why does `statsmodels` refuse to fit a seasonal Holt-Winters model on a series shorter than two full seasonal cycles?
4. What is the practical difference between additive and multiplicative seasonality, and which one guarantees the forecast can never go negative?
5. Why does damping the trend (`damped_trend=True`) matter more the further out your forecast horizon goes?
6. In a demand regression, why is `log(price)` usually preferred over raw price as a feature?
7. Define collinearity in one sentence, and name the diagnostic statistic this week used to detect it.
8. A regression coefficient is statistically significant (p < 0.001) but has the "wrong" sign for the business. Is that model still trustworthy? Why or why not?
9. What's the difference between a model's fit quality (R², holdout MAPE) and the interpretability of its individual coefficients — can one be good while the other is bad?
10. What is "lookahead bias," and name the one-line coding mistake this week identified as its most common cause.
11. Why is shuffled `KFold` cross-validation the wrong tool for a time series, even setting lag features aside entirely?
12. Define rolling-origin (walk-forward) backtesting in your own words.
13. Name one structural reason a gradient-boosted tree ensemble cannot extrapolate a trend the way a linear regression can.
14. Why is mean signed error (bias) worth reporting alongside MAE, not instead of it?
15. What does "intermittent demand" mean, and roughly what nonzero-period percentage did this week's example SKU have?
16. In Croston's method, what are the two quantities smoothed separately, and how are they combined into a forecast?
17. Why is MAPE a poor metric for scoring an intermittent-demand forecast?
18. Define "incoherence" in a forecast hierarchy.
19. Contrast bottom-up and top-down hierarchical forecasting in one sentence each.
20. Name the modern reconciliation method mentioned this week that doesn't require picking one level as "ground truth," and describe in one sentence what it optimizes for instead.

---

## Problem 2 — Hand-run three steps of Holt-Winters level/trend updates (60 min)

Simplify to a **non-seasonal** damped Holt (Holt-Winters without the seasonal term — sometimes called "damped trend exponential smoothing") so you can trace the arithmetic by hand. Given:

```
alpha = 0.3, beta = 0.2, phi (damping) = 0.9
Starting values: L_0 = 100, T_0 = 4
Observed:        y_1 = 108, y_2 = 111, y_3 = 118
```

Update equations (no seasonal term):

```
L_t = alpha * y_t + (1 - alpha) * (L_{t-1} + phi * T_{t-1})
T_t = beta  * (L_t - L_{t-1}) + (1 - beta) * phi * T_{t-1}
Forecast for period t+1 made at time t:  yhat_{t+1} = L_t + phi * T_t
```

In `holt-by-hand.md`, show your work for all three steps:

1. Compute `L_1`, `T_1`, and the one-step-ahead forecast `yhat_2` made at time 1. Compare `yhat_2` to the actual `y_2 = 111` — how far off was it?
2. Compute `L_2`, `T_2`, and `yhat_3`. Compare to `y_3 = 118`.
3. Compute `L_3`, `T_3`, and the forecast for period 4, `yhat_4`.
4. In one sentence, explain what would change in your arithmetic if `phi` were `1.0` instead of `0.9` (i.e., no damping) — which terms would be affected, and would the period-4 forecast be higher or lower?

*(Check your Step 1 arithmetic: `L_1` should come out to `104.9`, and `yhat_2` should be noticeably below the actual `111` — that gap is exactly the kind of lag a low `alpha` produces, the same phenomenon Lecture 1 describes SES having on trended data.)*

---

## Problem 3 — Diagnose a regression coefficient table (60 min)

You're handed this coefficient table from a colleague's demand regression, with no other context, and asked to review it before it ships to the merchandising team:

| Feature | Coefficient | p-value | VIF |
|---|---:|---:|---:|
| `const` | 850.2 | 0.000 | — |
| `t` (trend) | 0.41 | 0.000 | 1.1 |
| `log_price` | +28.6 | 0.000 | 187.4 |
| `discount_pct` | +61.3 | 0.000 | 203.9 |
| `holiday_flag` | 12.1 | 0.31 | 1.2 |
| `sin52` | 65.0 | 0.000 | 1.1 |

In `regression-review.md`, answer:

1. Which coefficient(s) should stop you before you approve this table for use? Name the specific number(s) and why.
2. What is the most likely relationship between `log_price` and `discount_pct` in the underlying data, given what you see in this table? (You don't have the raw data — reason from the VIF and coefficient signs alone, the way a reviewer would in a real code/model review.)
3. Propose a specific fix — which column would you drop or combine, and why that one rather than the other?
4. `holiday_flag` has a p-value of 0.31 — is it safe to just delete this feature from the model? What would you check before deciding, beyond the p-value alone?
5. If, after your fix, the holdout MAPE is nearly identical to the original broken model's holdout MAPE, what does that tell you — and what does it *not* tell you — about which fix was "better"?

---

## Problem 4 — Critique three backtest designs (60 min)

Three junior analysts each designed a backtest for the same 104-week series and reported their model's accuracy. In `backtest-critique.md`, for each one: **state whether it leaks, and if so, exactly how.**

**Analyst A:**
```python
from sklearn.model_selection import train_test_split
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, shuffle=True, random_state=1)
```

**Analyst B:**
```python
# "rolling" backtest — one fold
train = df.iloc[:80]
test = df.iloc[80:94]
# lag features were built like this, before the split:
df["roll_mean_4"] = df["units"].rolling(4).mean()
```

**Analyst C:**
```python
# 3 folds, walking forward, refit each time
for train_end in [60, 73, 86]:
    train = df.iloc[:train_end]
    test = df.iloc[train_end:train_end+13]
    model = fit(train)
    score(model, test)
# lag features built as: df["roll_mean_4"] = df["units"].shift(1).rolling(4).mean()
```

For each analyst: (a) does it leak, (b) if so, which specific line causes it and why, (c) what number would you expect to see — falsely optimistic, falsely pessimistic, or roughly honest — as a result.

---

## Problem 5 — Croston by hand (45 min)

A slow-moving spare part sold these units over 10 weeks: `[0, 0, 3, 0, 0, 0, 2, 0, 0, 4]`.

Using Croston's method with `alpha = 0.2`:

1. Identify the first nonzero observation, and initialize `z` (demand size) and `p` (interval) from it.
2. Walk forward week by week. At each nonzero observation, update both `z` and `p`; at each zero observation, only increment the periods-since-last-demand counter `q`.
3. Report the final `z`, `p`, and forecast rate (`z/p`) after all 10 weeks, in `croston-by-hand.md`, showing your work at each update step (not just the final answer).
4. Compare your hand-computed rate to the plain average of all 10 weeks (`sum/10`). State which is higher and, in one sentence, why they're not the same number even though both are "average demand" in some sense.

## Submission

Commit all five files to your portfolio under `c40-week-04/homework/`.
