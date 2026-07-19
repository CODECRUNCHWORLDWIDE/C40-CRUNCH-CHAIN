# Week 4 — Quiz

Fifteen questions. Lectures closed. Aim for 12/15 before starting Week 5. A mix of multiple-choice and short computation/interpretation — the answer key explains the *why*, not just the letter.

---

**Q1.** Holt-Winters triple exponential smoothing tracks three components. Which set is correct?

- A) Level, seasonality, noise
- B) Level, trend, seasonal
- C) Trend, cycle, irregular
- D) Mean, variance, autocorrelation

---

**Q2.** A fitted Holt-Winters model comes back with `gamma ≈ 0`. What is the most defensible interpretation?

- A) The series has no seasonality at all — drop the seasonal term.
- B) The fit failed and should be discarded.
- C) The seasonal *shape* is stable year over year, so little benefit comes from updating it further with new observations — check the seasonal component values themselves before concluding there's no seasonality.
- D) `gamma` should always be forced to at least 0.1.

---

**Q3.** Why does `statsmodels` raise an error fitting seasonal Holt-Winters (`seasonal_periods=52`) on a 70-week series?

- A) 70 is not divisible by 52.
- B) There's less than two full seasonal cycles of history, so trend and season can't be statistically separated.
- C) Weekly data always requires `seasonal_periods=7`.
- D) `ExponentialSmoothing` requires at least 200 observations regardless of seasonality.

---

**Q4.** In a demand regression, why prefer `log(price)` over raw price as a feature?

- A) `statsmodels` cannot fit models with raw currency values.
- B) The resulting coefficient is comparable across SKUs at different price points, reading roughly as "% demand change per 1% price change."
- C) Log-transforming always improves R².
- D) It removes the need for a holdout set.

---

**Q5.** A regression's `log_price` coefficient comes back **positive** (more price, more demand) with `p < 0.001`. What's the right next step?

- A) Ship it — statistical significance means it's correct.
- B) Increase prices immediately based on this finding.
- C) Check for collinearity (e.g., with a promo dummy) before trusting the sign — a significant coefficient with an implausible sign is a red flag, not a strong result.
- D) Re-run the regression with a different random seed until the sign flips.

---

**Q6.** What does an infinite (or extremely large) VIF on two features most directly indicate?

- A) The two features are perfectly (or near-perfectly) linearly predictable from each other — collinearity.
- B) The model has too few observations.
- C) One of the features has outliers.
- D) The target variable is not normally distributed.

---

**Q7.** You compute a rolling 4-week average feature as `df["units"].rolling(4).mean()` and use it to forecast `df["units"]`. What's wrong?

- A) Nothing — this is the standard way to build a rolling feature.
- B) `rolling(4)` should be `rolling(5)`.
- C) The window includes the current row, so the feature partially contains the answer you're trying to predict — lookahead bias. It should be `df["units"].shift(1).rolling(4).mean()`.
- D) Rolling means can't be used with `GradientBoostingRegressor`.

---

**Q8.** Why is shuffled `KFold` cross-validation inappropriate for a time series forecasting problem, even if no lag features are involved?

- A) `KFold` doesn't work with `pandas` DataFrames.
- B) Adjacent time periods are autocorrelated, so a randomly-selected test fold is statistically similar to nearby training rows — not a fair test of forecasting genuinely unseen future data.
- C) `KFold` only supports classification problems.
- D) It's actually fine; the concern only applies to lag features.

---

**Q9.** Describe rolling-origin (walk-forward) backtesting in one sentence.

*(Short answer — write your own definition before checking the key.)*

---

**Q10.** A gradient-boosted tree ensemble trained on data where weekly units ranged from 128 to 317 is asked to forecast a week where the true value turns out to be 339. What will the model's prediction most likely do?

- A) Correctly predict 339, since boosting corrects errors iteratively.
- B) Predict a value at or below roughly 317 — trees cannot output a value outside the range of targets seen in training, so a still-trending series gets systematically underforecast at new highs.
- C) Predict a negative number due to overfitting.
- D) Raise a runtime error, since 339 wasn't in the training data.

---

**Q11.** Why should you report **mean signed error (bias)** alongside MAE when evaluating a backtest?

- A) Bias is just MAE calculated differently, so it's redundant but harmless to include.
- B) A low-MAE model can still have a consistent directional bias (e.g., always underforecasting peaks) that matters operationally even though the average error looks small.
- C) MAE cannot be computed on time series data.
- D) Bias is only relevant for classification models.

---

**Q12.** Roughly what fraction of periods were **nonzero** in this week's intermittent-demand example SKU?

- A) About 95%
- B) About 65%
- C) About 36%
- D) About 5%

---

**Q13.** In Croston's method, what two quantities are smoothed separately, and how are they combined?

*(Short answer.)*

---

**Q14.** Why is plain MAPE a poor metric for scoring an intermittent-demand forecast?

- A) MAPE can only be computed for daily data, not weekly.
- B) MAPE is undefined (division by zero) whenever the actual value is 0, which is common in intermittent series — MASE or a similar scaled metric is the standard alternative.
- C) MAPE always favors the model that predicts zero.
- D) MAPE cannot be computed in Python.

---

**Q15.** A company forecasts each SKU independently and separately forecasts the company-wide total. The sum of the SKU forecasts doesn't match the total forecast. What is this problem called, and name one of the two simple fixes this week covered?

*(Short answer.)*

---

## Answer key

**Q1 — B.** Level, trend, seasonal — the three components Holt-Winters tracks with separate smoothing weights (`alpha`, `beta`, `gamma`).

**Q2 — C.** A near-zero `gamma` means the optimizer found little benefit in continuing to update an already-good seasonal estimate — it's evidence of a *stable* seasonal pattern, not necessarily *no* seasonal pattern. Always check the seasonal component values (not just the weight) before concluding there's no seasonality.

**Q3 — B.** Two full seasonal cycles are the statistical minimum needed to separate a structural trend from a repeating seasonal pattern; with less, the heuristic initializer has no way to tell them apart, and `statsmodels` refuses rather than silently guessing.

**Q4 — B.** Log-price coefficients are comparable across SKUs and read approximately as a percentage demand response per percentage price change (a semi-elasticity) — far more interpretable and transferable than a raw-dollar coefficient.

**Q5 — C.** Statistical significance says the coefficient is precisely estimated, not that it's causally correct — check collinearity (VIF, correlation) with other features before trusting an implausible sign. This week's price/promo example is exactly this trap.

**Q6 — A.** VIF measures how well a feature is predicted by the other features in the model; a very large or infinite VIF means near-perfect (or perfect) linear redundancy — the model literally cannot separate the two features' individual effects.

**Q7 — C.** Forgetting `.shift(1)` before `.rolling()` is the single most common lookahead-bias bug in ML forecasting — the "feature" ends up containing the target's own current-period value.

**Q8 — B.** Time series rows are not independent and identically distributed — nearby periods are correlated by nature (shared trend/season), so a random train/test split tests the model on data that's statistically similar to what it trained on, inflating reported accuracy.

**Q9.** *Sample answer:* Repeatedly slide a cutoff point ("origin") forward through the series; at each origin, train only on data strictly before it, forecast a fixed horizon after it, and score that forecast — so every fold respects the rule that the model never sees data from its own future.

**Q10 — B.** Tree-based models predict by averaging training-target values that land in a leaf, so they can never output a value outside the training target's observed range — a structural limitation, not a tuning problem. This week's real backtest showed exactly this pattern on a trending SKU (training max 317, actual peak 339, model capped near/below 306).

**Q11 — B.** MAE alone can hide a systematic direction to the error (e.g., "always 6 units low at the peak") that matters a great deal for inventory decisions even when the average magnitude of error looks acceptable.

**Q12 — C.** About 36% of weeks were nonzero for the intermittent SKU (`SPR-901`) in this week's data — the other ~64% were flat zero, the defining shape of intermittent demand.

**Q13.** *Sample answer:* Croston smooths the **demand size** (`z`, only updated on nonzero periods) and the **inter-demand interval** (`p`, periods between nonzero demands, also only updated on nonzero periods) separately, using the same exponential-smoothing weight for both; the forecast is `z / p`, held constant until the next nonzero observation updates both.

**Q14 — B.** MAPE divides by the actual value, so any period with zero actual demand makes the calculation undefined — a fatal flaw for a series that's mostly zeros. MASE (mean absolute scaled error) or similar scale-free metrics that don't divide by the raw actual are the standard fix.

**Q15.** *Sample answer:* This is **incoherence** in a forecast hierarchy. The two simple fixes covered this week are **bottom-up** (forecast every leaf, sum for the total — coherent by construction) and **top-down** (forecast only the total, split to leaves by a fixed historical proportion — also coherent by construction, but can badly misrepresent a leaf whose seasonal shape diverges from the aggregate's).
