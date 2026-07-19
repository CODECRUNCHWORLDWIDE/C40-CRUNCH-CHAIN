# Exercise 2 — Build a Moving-Average Forecast

**Goal:** Turn Lecture 2's SQL window-function forecasts into reusable pandas functions, and confirm — with your own code, on your own machine — the exact numbers the lecture showed you for the Alpine Shell Jacket.

**Estimated time:** 90 minutes.

## Setup

Start from `pull_demand.py` (Exercise 1). Create `forecasts.py` and import your puller:

```python
from pull_demand import pull_demand
import pandas as pd
import numpy as np

alpine = pull_demand("JCK-ALP-001").set_index("week_start").asfreq("W-MON")
```

## Tasks

### Task 1 — Naive forecast

```python
def naive_forecast(series: pd.Series) -> pd.Series:
    """One-step-ahead naive forecast: forecast(t) = actual(t-1)."""
    return series.shift(1)

alpine["naive"] = naive_forecast(alpine["units_sold"])
print(alpine[["units_sold", "naive"]].tail(5))
```

`.shift(1)` is pandas' equivalent of SQL's `LAG(x, 1)` — same idea, different tool. The first row of `naive` will be `NaN` — there's no prior week to forecast from. That's correct, not a bug; leave it `NaN` rather than filling it with something arbitrary.

### Task 2 — Seasonal naive forecast

```python
def seasonal_naive_forecast(series: pd.Series, period: int = 52) -> pd.Series:
    """Forecast(t) = actual(t - period)."""
    return series.shift(period)

alpine["seasonal_naive"] = seasonal_naive_forecast(alpine["units_sold"])
print(alpine[["units_sold", "seasonal_naive"]].tail(5))
```

**Expected:** the first 52 rows of `seasonal_naive` are all `NaN` — there's no "same week last year" until week 53. Confirm: `alpine["seasonal_naive"].isna().sum()` should be exactly `52`.

### Task 3 — Moving-average forecast (with the leakage check from the lecture)

```python
def moving_average_forecast(series: pd.Series, window: int = 4) -> pd.Series:
    """Forecast(t) = mean of the `window` periods strictly BEFORE t. No leakage."""
    return series.shift(1).rolling(window=window).mean()

alpine["ma4"] = moving_average_forecast(alpine["units_sold"], window=4)
print(alpine[["units_sold", "ma4"]].tail(5))
```

Notice the `.shift(1)` **before** `.rolling(...)`. This is the pandas version of the lecture's `ROWS BETWEEN 4 PRECEDING AND 1 PRECEDING` — shift first so the rolling window never includes the value you're trying to forecast. Prove to yourself this matters:

```python
# THE BUG: rolling without shifting first — this leaks the current value into its own forecast
leaky = alpine["units_sold"].rolling(window=4).mean()
print((leaky != alpine["ma4"]).sum())   # should be > 0 — the two are NOT the same series
```

### Task 4 — Reproduce the lecture's exact numbers

Slice out the same 12-week holdout Lecture 2 used (`2024-10-07` through `2024-12-23`) and confirm your MA4 forecasts match the lecture's table to the first decimal:

```python
holdout = alpine.loc["2024-10-07":"2024-12-23"]
print(holdout[["units_sold", "naive", "seasonal_naive", "ma4"]])
```

**Expected — first two rows:**

| week_start | units_sold | naive | seasonal_naive | ma4 |
|---|---:|---:|---:|---:|
| 2024-10-07 | 138 | 107 | 136 | 114.5 |
| 2024-10-14 | 142 | 138 | 138 | 122.75 |

If your `ma4` column doesn't match, the most common cause is forgetting the `.shift(1)` before `.rolling()` — go back and check Task 3.

### Task 5 — Build the same three forecasts for all six SKUs, in one loop

```python
from pull_demand import pull_all_demand

all_demand = pull_all_demand()
results = {}
for sku_id, group in all_demand.groupby("sku_id"):
    s = group.set_index("week_start")["units_sold"].asfreq("W-MON")
    df = pd.DataFrame({"units_sold": s})
    df["naive"] = naive_forecast(s)
    df["seasonal_naive"] = seasonal_naive_forecast(s)
    df["ma4"] = moving_average_forecast(s, window=4)
    results[sku_id] = df

print(results["SAN-TRL-040"].tail(5))
```

## Done when…

- [ ] `naive_forecast`, `seasonal_naive_forecast`, and `moving_average_forecast` all live in `forecasts.py` and are correctly leak-free.
- [ ] `alpine["seasonal_naive"].isna().sum() == 52`.
- [ ] Your MA4 numbers for the 12-week Alpine holdout match the lecture's table.
- [ ] You've proven to yourself (Task 3's `leaky` comparison) that omitting `.shift(1)` before `.rolling()` genuinely produces a different, leaked series.
- [ ] `results` holds all six SKUs' forecasts in one dict of DataFrames.

## Stretch

- Add a 13-week moving average (`window=13`, a quarter) alongside the 4-week one. On the Alpine Jacket's winter-ramp holdout, does the wider window lag *more* or *less* than MA4? Explain why, referencing Lecture 2 Section 4.
- For `SAN-TRL-040` (Trail Sandal), print the seasonal-naive forecast for the weeks where `units_sold` is 0. What does seasonal naive predict for those weeks, and is that a reasonable forecast given what you know about this SKU from Lecture 1?

## Submission

Commit `forecasts.py` to your portfolio under `c40-week-03/exercise-02/`.
