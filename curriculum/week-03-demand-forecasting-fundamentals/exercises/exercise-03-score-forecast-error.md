# Exercise 3 — Score Forecast Error vs. a Baseline

**Goal:** Implement MAE, MAPE, RMSE, and bias as reusable functions, score every method from Exercise 2 plus single/double exponential smoothing, and produce the master comparison table from Lecture 3 — with your own code, not copied numbers.

**Estimated time:** 75 minutes.

## Setup

Continue from `forecasts.py`. Create `score_errors.py`.

## Tasks

### Task 1 — Implement the four metrics

```python
import numpy as np
import pandas as pd

def mae(actual: pd.Series, forecast: pd.Series) -> float:
    err = (actual - forecast).dropna()
    return err.abs().mean()

def rmse(actual: pd.Series, forecast: pd.Series) -> float:
    err = (actual - forecast).dropna()
    return np.sqrt((err ** 2).mean())

def mape(actual: pd.Series, forecast: pd.Series) -> float:
    """Mean absolute percentage error. Rows where actual == 0 are EXCLUDED
    and the exclusion count is reported — see Lecture 3, Section 3, on why
    MAPE cannot be computed at a zero actual."""
    df = pd.DataFrame({"a": actual, "f": forecast}).dropna()
    zero_mask = df["a"] == 0
    n_excluded = zero_mask.sum()
    df = df[~zero_mask]
    pct_err = (df["a"] - df["f"]).abs() / df["a"]
    return pct_err.mean() * 100, n_excluded

def bias(actual: pd.Series, forecast: pd.Series) -> float:
    err = (actual - forecast).dropna()
    return err.mean()
```

Note `mape` returns a **tuple**: the percentage *and* how many rows it had to drop. Never report a MAPE without also reporting (or at least checking) that second number — a MAPE computed by silently dropping half your holdout is not the same claim as one computed on all of it.

### Task 2 — Add single and double exponential smoothing to your forecast set

```python
def ses_forecast(series: pd.Series, alpha: float, init_periods: int = 8) -> pd.Series:
    """One-step-ahead SES forecast, seeded on the mean of the first `init_periods`."""
    values = series.to_numpy(dtype=float)
    level = values[:init_periods].mean()
    fcs = np.empty(len(values))
    for i, v in enumerate(values):
        fcs[i] = level                      # forecast BEFORE seeing v
        level = alpha * v + (1 - alpha) * level
    return pd.Series(fcs, index=series.index)

def holt_forecast(series: pd.Series, alpha: float, beta: float, init_periods: int = 4) -> pd.Series:
    """One-step-ahead Holt (double exponential smoothing) forecast."""
    values = series.to_numpy(dtype=float)
    level = values[0]
    trend = (values[init_periods - 1] - values[0]) / (init_periods - 1)
    fcs = np.empty(len(values))
    for i, v in enumerate(values):
        fcs[i] = level + trend              # forecast BEFORE seeing v
        new_level = alpha * v + (1 - alpha) * (level + trend)
        new_trend = beta * (new_level - level) + (1 - beta) * trend
        level, trend = new_level, new_trend
    return pd.Series(fcs, index=series.index)
```

Both functions carefully record the forecast **before** updating on the current observation — the same one-step-ahead discipline as the moving-average leakage check in Exercise 2. If you update first and forecast second, you've leaked the current actual into its own forecast again.

### Task 3 — Reproduce Lecture 3's master comparison table

```python
from pull_demand import pull_demand
from forecasts import naive_forecast, seasonal_naive_forecast, moving_average_forecast

alpine = pull_demand("JCK-ALP-001").set_index("week_start").asfreq("W-MON")["units_sold"]

methods = {
    "ses_0.2": ses_forecast(alpine, alpha=0.2),
    "ses_0.4": ses_forecast(alpine, alpha=0.4),
    "ma4": moving_average_forecast(alpine, window=4),
    "naive": naive_forecast(alpine),
    "holt_0.3_0.2": holt_forecast(alpine, alpha=0.3, beta=0.2),
    "seasonal_naive": seasonal_naive_forecast(alpine),
}

holdout_actual = alpine.loc["2024-10-07":"2024-12-23"]

rows = []
for name, fc in methods.items():
    fc_holdout = fc.loc["2024-10-07":"2024-12-23"]
    m, n_excl = mape(holdout_actual, fc_holdout)
    rows.append({
        "method": name,
        "MAE": round(mae(holdout_actual, fc_holdout), 2),
        "RMSE": round(rmse(holdout_actual, fc_holdout), 2),
        "MAPE": round(m, 2),
        "Bias": round(bias(holdout_actual, fc_holdout), 2),
    })

scorecard = pd.DataFrame(rows).sort_values("MAE")
print(scorecard.to_string(index=False))
```

**Expected:** your `scorecard`, sorted by MAE ascending, should match Lecture 3's master table — seasonal naive and Holt at the top, SES(0.2) at the bottom. If your numbers are close but not exact, check your seed windows (`init_periods`) match the lecture's.

### Task 4 — Score a different SKU and watch the ranking change

Run the exact same scoring loop against `BAG-DAY-020` (Daypack — no seasonality) instead of the Alpine Jacket.

```python
daypack = pull_demand("BAG-DAY-020").set_index("week_start").asfreq("W-MON")["units_sold"]
# ... repeat Task 3's loop with `daypack` in place of `alpine`
```

**Expected pattern (not exact numbers — compute your own):** seasonal naive should do noticeably *worse* relative to naive/MA here than it did on the Alpine Jacket — the Daypack has almost no seasonal signal, so "same week last year" isn't a meaningfully better forecast than "last week," it's just adding a full year of extra noise to the guess. Write one sentence in a comment explaining, in your own words, why this ranking flip happened.

### Task 5 — Confirm the MAPE zero-division handling on the Trail Sandal

```python
sandal = pull_demand("SAN-TRL-040").set_index("week_start").asfreq("W-MON")["units_sold"]
sn_forecast = seasonal_naive_forecast(sandal)
m, n_excl = mape(sandal, sn_forecast)
print(f"MAPE: {m:.2f}%, rows excluded for zero actual: {n_excl}")
```

**Expected:** `n_excl` should be greater than 0 — the Trail Sandal genuinely has weeks with `units_sold == 0` (you found them in Exercise 1's stretch goal). If you skipped that stretch goal, do it now — you cannot honestly report a MAPE on this SKU without knowing how many weeks it silently dropped.

## Done when…

- [ ] `mae`, `rmse`, `mape` (with exclusion count), and `bias` are all implemented and correctly handle `NaN`s from the shift/rolling warm-up period.
- [ ] `ses_forecast` and `holt_forecast` are one-step-ahead and leak-free.
- [ ] Your Alpine Jacket scorecard matches Lecture 3's master table.
- [ ] Your Daypack scorecard shows a different ranking, and you've written the one-sentence explanation.
- [ ] Your Trail Sandal MAPE run reports a nonzero exclusion count.

## Stretch

- Add RMSE-to-MAE ratio as a column to your scorecard (`RMSE / MAE`). Which method has the largest ratio, and what does that tell you about the *shape* of its errors (a few big misses vs. many uniform ones)?
- Tune `alpha` for SES from 0.1 to 0.9 in steps of 0.1 on the Alpine Jacket holdout and plot (or just print) MAE against alpha. Is the relationship monotonic, or is there a sweet spot?

## Submission

Commit `score_errors.py` and both printed scorecards (Alpine Jacket, Daypack) to your portfolio under `c40-week-03/exercise-03/`.
