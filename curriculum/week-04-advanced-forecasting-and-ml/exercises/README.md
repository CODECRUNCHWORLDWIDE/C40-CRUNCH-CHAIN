# Week 4 Exercises — Overview

Three exercises, one model family each, all against the same `weekly_demand` table so results are directly comparable by the time you reach the mini-project.

| # | Exercise | Model | Builds on |
|--:|----------|-------|-----------|
| 1 | [exercise-01-fit-holt-winters.md](./exercise-01-fit-holt-winters.md) | Holt-Winters triple exponential smoothing | Lecture 1 |
| 2 | [exercise-02-regression-with-promo-features.md](./exercise-02-regression-with-promo-features.md) | Linear regression on price/promo/calendar features | Lecture 2 |
| 3 | [exercise-03-rolling-origin-backtest.md](./exercise-03-rolling-origin-backtest.md) | Rolling-origin backtest loop, built by hand | Lecture 3 |

## How to work these

- Each exercise fits a model on the **same SKU** you'll use across all three (`TNT-220`, Northeast region — deliberately *not* the `JCK-100` example worked in the lectures, so you're applying the method, not copying the printed numbers). The lectures walk through `JCK-100`; you'll do the analogous work on `TNT-220`, a summer-peaking, promo-responsive tent — same techniques, different shape, different numbers.
- Use the same **13-week holdout** convention as Week 3 and this week's lectures, unless an exercise says otherwise.
- Write real code in a `.py` file or notebook per exercise — these are not multiple-choice. "Expected" sections give you a range to sanity-check against, not an exact number to reverse-engineer; different random seeds, slightly different feature choices, or a different `statsmodels` version can shift a number a little. Focus on whether your *method* is right, not on hitting a number to the decimal.
- Keep every `.py`/`.sql` file you write — you'll reuse the feature-engineering and backtesting code directly in the mini-project.

## Setup (all three exercises)

```python
import pandas as pd, numpy as np
from sqlalchemy import create_engine

engine = create_engine("postgresql://localhost/crunch_chain")   # or sqlite:///crunch_chain.db
df = pd.read_sql(
    "SELECT week_start, units, unit_price, promo_flag, holiday_flag "
    "FROM weekly_demand WHERE sku = 'TNT-220' AND region = 'Northeast' "
    "ORDER BY week_start",
    con=engine,
)
```

If you haven't loaded `weekly_demand` yet, run the mini-project's [`generate_demand.py`](../mini-project/README.md) first — it's the same seed data every file in this week depends on.

## Submission

Commit all three `.py` files (or one notebook per exercise) to your portfolio under `c40-week-04/exercise-0N/`, same convention as prior weeks.
