# Week 3 — Quiz

Fifteen questions. Lectures closed. Aim for 13/15 before starting Week 4. A mix of multiple-choice and short "what does this compute?" — the answer key at the bottom explains the *why*, not just the letter.

---

**Q1.** In demand decomposition, what's the key test for distinguishing trend from seasonality?

- A) Trend is always positive; seasonality can be negative.
- B) Seasonality repeats on a fixed calendar cycle and returns to the same relative position; trend is a sustained drift that does not reset.
- C) Trend only applies to slow-moving SKUs.
- D) There is no meaningful difference — they're the same thing.

<details>
<summary>Answer</summary>

**B** — the defining test is whether the pattern snaps back to the same relative position on a fixed cycle (seasonality) versus keeps drifting one direction with no reset (trend).

</details>

---

**Q2.** A SKU's seasonal swing grows from ±5 units when the yearly average was 50, to ±10 units when the yearly average doubled to 100. This is evidence for:

- A) An additive decomposition model
- B) A multiplicative decomposition model
- C) Pure noise, ignore it
- D) A calculation error — seasonality can't change size

<details>
<summary>Answer</summary>

**B** — a seasonal swing that scales proportionally with the level (constant percentage, growing absolute size) is the signature of a multiplicative model.

</details>

---

**Q3.** Why does a centered moving average trend estimate have missing values at both ends of a series?

- A) It's a bug in the software
- B) The first and last observations are always corrupted
- C) There aren't enough neighboring periods on one side to center the window
- D) Centered moving averages only work on even-length series

<details>
<summary>Answer</summary>

**C** — a centered window needs `period/2` neighbors on each side; the first and last `period/2` points don't have enough neighbors on one side, so no trend value can be computed there.

</details>

---

**Q4.** The naive forecasting method's rule is:

- A) `ŷ_{t+1} = average of all history`
- B) `ŷ_{t+1} = y_t` (tomorrow looks like today)
- C) `ŷ_{t+1} = y_{t-52}` (tomorrow looks like this time last year)
- D) `ŷ_{t+1} = 0`

<details>
<summary>Answer</summary>

**B** — naive forecasts tomorrow as exactly today's value.

</details>

---

**Q5.** Seasonal naive forecasting requires:

- A) No history at all — it works from day one
- B) At least one full seasonal cycle of history before it can produce its first forecast
- C) A trend estimate
- D) Multiplicative decomposition specifically

<details>
<summary>Answer</summary>

**B** — seasonal naive looks up the same period one cycle back, so it needs at least one full cycle of history to exist before it can produce a first forecast.

</details>

---

**Q6.** In SQL, `AVG(units_sold) OVER (ORDER BY week_start ROWS BETWEEN 4 PRECEDING AND 1 PRECEDING)` computes:

- A) A 5-week average including the current week — this is a leakage bug
- B) A 4-week average of the periods strictly BEFORE the current week — correctly leak-free
- C) The average of the whole series
- D) A syntax error; `PRECEDING` can't be used twice

<details>
<summary>Answer</summary>

**B** — `4 PRECEDING AND 1 PRECEDING` is four rows ending one row before the current one — the current row's own value is correctly excluded, avoiding leakage.

</details>

---

**Q7.** Why does a k-period moving average structurally lag a series with a sustained upward trend?

- A) It doesn't — moving averages track trend perfectly
- B) Its forecast is built from `k` past values whose average age is roughly `(k+1)/2` periods, so it's always "looking backward" relative to a rising series
- C) SQL window functions are inherently slower than the trend
- D) It only lags if `k` is even

<details>
<summary>Answer</summary>

**B** — the average age of the k values behind any moving-average forecast is `(k+1)/2` periods, which is why it always lags a steadily moving series.

</details>

---

**Q8.** On the Alpine Shell Jacket's winter-ramp holdout (Lecture 2/3), which method had the lowest MAE using the lectures' default parameters?

- A) SES with α=0.2
- B) 4-week moving average
- C) Seasonal naive
- D) Naive

<details>
<summary>Answer</summary>

**C** — seasonal naive had the lowest MAE (7.67) in the lecture's default-parameter comparison table, beating even Holt's default settings.

</details>

---

**Q9.** Single exponential smoothing's core limitation is:

- A) It requires two full years of history
- B) It has no concept of trend — it smooths around the current level but never adjusts its "aim" for sustained directional movement
- C) It can only be computed in Python, not SQL
- D) It always overforecasts

<details>
<summary>Answer</summary>

**B** — SES has one smoothed quantity (the level) and no trend term, so it can never anticipate sustained directional movement, only react to it after the fact.

</details>

---

**Q10.** What does Holt's method (double exponential smoothing) add on top of single exponential smoothing?

- A) A second, independently smoothed estimate of the trend, added to the level to form the forecast
- B) A second seasonal index
- C) Nothing — Holt's method and SES are the same algorithm
- D) A requirement for 52 weeks of history

<details>
<summary>Answer</summary>

**A** — Holt's method adds a separately smoothed trend estimate (controlled by β) that gets added to the level to project the forecast forward.

</details>

---

**Q11.** MAPE is undefined (or explodes toward infinity) when:

- A) The forecast is exactly correct
- B) The actual value for that period is zero
- C) The forecast is negative
- D) The series has a trend

<details>
<summary>Answer</summary>

**B** — dividing by an actual of zero is undefined; MAPE breaks down (or must exclude) any period where the actual is exactly zero.

</details>

---

**Q12.** A forecast has MAE = 12 and bias = +12 (every single error in the sample happens to be positive and roughly the same size). What does this combination tell you?

- A) The forecast is unbiased and just has moderate random error
- B) The forecast is systematically UNDER-forecasting — actuals are consistently coming in above the forecast
- C) The forecast is systematically OVER-forecasting
- D) MAE and bias can never be equal; this is impossible

<details>
<summary>Answer</summary>

**B** — a large positive bias with matching MAE means every error is in the same direction (actual > forecast), i.e., the method is systematically under-forecasting, not just randomly imprecise.

</details>

---

**Q13.** Why does RMSE penalize a single large miss more than MAE does, for the same overall error budget?

- A) RMSE ignores small errors entirely
- B) RMSE squares each error before averaging, so a error twice as large contributes four times as much to the sum, not twice
- C) RMSE is just MAE multiplied by a constant
- D) They penalize errors identically; the names are the only difference

<details>
<summary>Answer</summary>

**B** — squaring an error of 2x produces 4x the contribution to the sum (since (2e)² = 4e²), which is why RMSE reacts more strongly to a few large misses than MAE does.

</details>

---

**Q14.** On the Daypack (`BAG-DAY-020`, no meaningful seasonality) 12-week holdout, compared to naive and moving average, seasonal naive typically:

- A) Performs about the same, since seasonal naive always matches naive
- B) Performs noticeably WORSE — with no real seasonal signal, "same week last year" just adds a year of extra noise instead of a genuine calendar advantage
- C) Performs perfectly, since seasonal naive is always the best method
- D) Cannot be computed for a SKU with no seasonality

<details>
<summary>Answer</summary>

**B** — without real seasonal signal, looking up "last year's same week" just imports an extra year of unrelated noise instead of adding genuine predictive information, so it typically underperforms naive/MA on a flat, non-seasonal SKU.

</details>

---

**Q15.** What is the single rule this entire week insists you follow before shipping any forecasting method, no matter how sophisticated?

- A) Always use the method with the most tunable parameters
- B) Always prefer multiplicative decomposition over additive
- C) Always compare its scored error against a naive benchmark on held-out data before trusting it
- D) Always use exactly a 4-week moving average as a sanity check

<details>
<summary>Answer</summary>

**C** — the week's central thesis: score every method against a naive benchmark on held-out data before trusting it, regardless of how sophisticated the method is.

</details>

**Scoring:** 13+ → start Week 4. 10–12 → re-read the lecture sections behind your misses. <10 → re-read all three lectures from the top; Week 4's backtesting and ML methods assume this week's error metrics and baselines are already automatic.

---
