# Challenge 1 — Intermittent Demand with Croston

**Time:** ~90 minutes. **Difficulty:** Medium.

## The scenario

`SPR-901` (Repair Patch Kit) is a real shape of demand every operations team eventually has to plan for: a spare part, an accessory, a low-velocity SKU that most weeks sells **zero units** and occasionally sells a handful. In the Northeast region, `SPR-901` is nonzero in only about **36% of weeks** across three years of history — the other ~64% are flat zero.

Fit Holt-Winters or a moving average on this series and you get a forecast that's technically a number but tells the planner almost nothing useful: a flat "0.7 units/week" from a moving average blends "usually zero" and "sometimes 4" into a value the series basically never actually takes. **Croston's method** exists specifically for this shape: instead of smoothing the raw series, it separately tracks *how big* the demand is when it happens, and *how often* it happens — and combines the two into a rate estimate that answers the question a planner actually has.

## Background: how Croston's method works

Split the series into two derived series, updated only on periods with nonzero demand:

- **Demand size** `z` — smoothed with exponential smoothing, updated *only* on the periods where demand occurred.
- **Inter-demand interval** `p` — the smoothed number of periods *between* nonzero demands, also updated only when a new nonzero demand arrives.

The forecast is simply `z / p` — "average size when it happens" divided by "average periods between happenings" — held constant until the next nonzero observation updates both components. Both `z` and `p` use the same smoothing parameter `alpha` (0.1 is a common default, same intuition as SES: how much to trust the newest data point vs. the smoothed history).

```python
import numpy as np

def croston(ts, alpha=0.1):
    ts = np.asarray(ts, dtype=float)
    n = len(ts)
    first = np.argmax(ts > 0)          # index of first nonzero observation
    z, p, q = ts[first], first + 1, 1  # initialize size, interval, periods-since-last
    rate = np.zeros(n)
    for t in range(first + 1, n):
        if ts[t] > 0:
            z = alpha * ts[t] + (1 - alpha) * z
            p = alpha * q + (1 - alpha) * p
            q = 1
        else:
            q += 1
        rate[t] = z / p if p > 0 else 0.0
    return rate
```

## Your task

1. **Pull the series.** `SPR-901`, Northeast region, all 156 weeks. Confirm the ~36% nonzero rate yourself — don't take the number above on faith, compute it.

2. **Hold out the last 13 weeks**, same convention as every other model this week.

3. **Run Croston on the training data.** Report the final smoothed rate (`z/p` at the end of training) as "units per week." Compare it to the training period's plain average (`train.mean()`). They should be close but not identical — explain in `notes.md` why Croston's rate and the plain average aren't the same computation even though they answer a similar-sounding question.

4. **Score three forecasts on the holdout** — Croston's constant rate, the plain training average (constant), and a 4-week moving average of the last 4 training weeks (constant) — using **MAE**, not MAPE. *(MAPE is undefined whenever an actual value is 0, which is most weeks here — this is itself worth a sentence in `notes.md`: name at least one metric, beyond MAE, that's designed to handle a mostly-zero holdout better, even if you don't implement it. MASE — mean absolute scaled error — is the standard answer; look it up and describe how it avoids the division-by-zero problem.)*

5. **Look honestly at which model wins on MAE — and question it.** Depending on exactly which 13 weeks land in your holdout, a simple 4-week moving average can come out *ahead* of Croston on raw MAE, purely because a mostly-zero holdout rewards any forecast that's already close to zero, regardless of whether that forecast came from a principled model or got lucky. If that happens in your run, don't paper over it — write one paragraph in `notes.md` making the case for *why a demand planner might still prefer Croston's estimate* even if it doesn't win on this particular holdout's MAE. (Hint: think about what each model's output is actually useful *for* — Croston's rate feeds directly into a reorder-point calculation next week, per Lecture 1 of Week 5; a moving average that happens to sit near zero this quarter doesn't give you anything to plan a reorder point around if next quarter looks different.)

6. **State the real limitation.** Croston has a known, documented bias: because the demand-size and interval smoothing don't correct for each other, Croston forecasts are provably biased slightly *high* on average versions of this exact demand pattern. Look up the name of at least one variant designed to correct this bias (Syntetos-Boylan Approximation is the standard answer) and describe in one sentence what it changes.

## Deliverable

`notes.md` covering all six tasks, plus `croston.py` with your implementation and scoring code.

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|-------:|---------------------|
| Correct Croston implementation | 30% | Rate updates only on nonzero periods; matches the reference formula |
| Honest metric discussion | 25% | Names MASE (or equivalent) and explains *why* MAPE fails here, not just that it does |
| Honest comparison | 25% | Reports the real MAE numbers, including if Croston "loses" — and explains why that doesn't necessarily mean Croston is the wrong choice |
| Limitation awareness | 20% | Names the known bias and a real correction method, doesn't overstate Croston as a perfect solution |

## Submission

Commit `croston.py` and `notes.md` to your portfolio under `c40-week-04/challenge-01/`.
