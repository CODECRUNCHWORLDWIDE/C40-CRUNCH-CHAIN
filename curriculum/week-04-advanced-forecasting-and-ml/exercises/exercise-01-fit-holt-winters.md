# Exercise 1 — Fit a Holt-Winters Model

**Goal:** Fit triple exponential smoothing on a real, trended-and-seasonal SKU, read the fitted parameters like a diagnostic instead of a black box, and score it against Week 3's seasonal-naive baseline on a proper holdout.

**Estimated time:** 90 minutes.

## Setup

Use `TNT-220` (Trailhead Tent 2P), Northeast region — **not** the `JCK-100` jacket worked in Lecture 1. Same technique, different SKU: this one peaks in *summer*, not winter, and its trend is much gentler.

```python
import pandas as pd
df = pd.read_sql(
    "SELECT week_start, units FROM weekly_demand "
    "WHERE sku = 'TNT-220' AND region = 'Northeast' ORDER BY week_start",
    con=engine,
)
s = df.set_index("week_start")["units"].asfreq("W-MON")
print(len(s))   # must print 156 — 3 full years, enough for 2+ seasonal cycles
```

## Tasks

1. **Plot it first.** Before fitting anything, plot `s`. Confirm by eye: is there a visible upward or downward trend? Where does it peak in the calendar year? Write two sentences describing the shape in `notes.md` — you should be able to predict roughly what Holt-Winters' `beta` and the sign of the seasonal peak will look like before you fit anything.

2. **Split the holdout.** Hold out the **last 13 weeks** as `test`; everything before is `train`. Confirm `len(train) == 143`.

3. **Fit Holt-Winters** with `trend="add"`, `damped_trend=True`, `seasonal="add"`, `seasonal_periods=52`, `initialization_method="estimated"`. Print `fit.params[["smoothing_level","smoothing_trend","smoothing_seasonal","damping_trend"]]`.

4. **Interpret each parameter in one sentence** in `notes.md`: what does the fitted `alpha` say about how fast the level adapts? What does `gamma` say about how stable the seasonal shape is year over year? Does `beta` match the trend strength you predicted in Task 1?

5. **Forecast 13 weeks** with `fit.forecast(13)` and compute MAPE against `test`, using the exact formula from Week 3:
   ```python
   mape = (abs((test.values - fc.values) / test.values)).mean() * 100
   ```

6. **Compare to the seasonal-naive baseline** (`s.shift(52)` over the same holdout window). Report both MAPEs side by side in `notes.md` and state, in one sentence, *why* one wins — don't just report the numbers, explain the mechanism (same reasoning shape as Lecture 1 §3).

7. **Break it on purpose.** Re-fit with only the last 60 weeks of history (`train.iloc[-60:]`) instead of the full 143. What happens? *(Expected: an error — you don't have two full seasonal cycles. Paste the exact error message into `notes.md` and explain in your own words why `statsmodels` refuses rather than silently fitting something wrong.)*

## Expected result (sanity range, not an exact target)

- Fitted `alpha` should land somewhere in the 0.05–0.20 range; `gamma` may land near zero — that's a valid, informative result (see Lecture 1 §2), not a failed fit.
- Holt-Winters MAPE should land somewhere in the **8–14%** range on this SKU.
- Holt-Winters should **beat** the seasonal-naive baseline (which lands roughly in the 13–18% range) — if yours doesn't, double check you used `damped_trend=True` and the correct 13-week holdout before concluding Holt-Winters lost.

## Done when…

- [ ] `notes.md` states the fitted `alpha`/`beta`/`gamma`/`phi` and interprets each in a sentence.
- [ ] `notes.md` reports both MAPEs (Holt-Winters and seasonal-naive) and explains the mechanism behind the winner.
- [ ] Task 7's error message is pasted verbatim, with your explanation of the "two seasonal cycles" requirement.
- [ ] Your code runs top-to-bottom without manual intervention.

## Stretch

- Fit `seasonal="mul"` instead of `"add"` and compare holdout MAPE. Which wins on this SKU, and does that match what you'd expect given whether the seasonal swing looks like a fixed number of units or a fixed percentage as the trend grows?
- Add `fit.simulate(nsimulations=13, repetitions=200)` and report a 90% prediction interval for week 13 of the forecast alongside the point estimate.

## Submission

Commit `holt_winters.py` and `notes.md` to your portfolio under `c40-week-04/exercise-01/`.
