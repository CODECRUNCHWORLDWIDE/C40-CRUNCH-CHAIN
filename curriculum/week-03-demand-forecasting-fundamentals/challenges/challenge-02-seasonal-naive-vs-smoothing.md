# Challenge 2 — Seasonal Naive vs. Smoothing Face-off

**Time:** ~60 minutes. **Difficulty:** Medium-Hard. **Has a real, verifiable answer — but also a judgment call on top of it.**

## The scenario

Lecture 3's master comparison table, on the Alpine Shell Jacket's 12-week winter-ramp holdout, had seasonal naive winning on MAE, RMSE, and MAPE, with Holt's default settings (α=0.3, β=0.2) trailing at MAE 8.72 vs. seasonal naive's 7.67. It would be easy to walk away thinking "seasonal naive just wins, full stop, when there's real seasonality." **That conclusion is wrong, and this challenge proves it to you with your own hands.**

Exponential smoothing's whole advantage over a fixed-window method is that its parameters can be *tuned*. Lecture 3 used one specific (α, β) pair, chosen to be reasonable, not chosen to be optimal. Your job: tune it properly, find out whether a well-tuned Holt can actually beat seasonal naive on this exact holdout, and then answer the harder question — **should you ship the tuned version, even if it wins?**

## Your task

### Part A — Grid search Holt's parameters (20 min)

Using your `holt_forecast` function from Exercise 3, score Holt on the Alpine Shell Jacket's same 12-week holdout (`2024-10-07` – `2024-12-23`) across a grid of α and β values:

```python
results = []
for alpha in [0.1, 0.2, 0.3, 0.4, 0.5]:
    for beta in [0.1, 0.2, 0.3, 0.5, 0.7, 0.9]:
        fc = holt_forecast(alpine, alpha=alpha, beta=beta)
        fc_holdout = fc.loc["2024-10-07":"2024-12-23"]
        results.append({
            "alpha": alpha, "beta": beta,
            "MAE": mae(holdout_actual, fc_holdout),
            "RMSE": rmse(holdout_actual, fc_holdout),
            "Bias": bias(holdout_actual, fc_holdout),
        })
grid = pd.DataFrame(results).sort_values("MAE")
print(grid.head(10))
```

Find the (α, β) pair with the lowest MAE. **Does it beat seasonal naive's MAE of 7.67?** Report the winning pair and its full scorecard (MAE, RMSE, bias).

### Part B — Explain *why* that particular β wins (20 min)

Look specifically at the winning `β` value, not just the winning MAE. β controls how fast Holt's **trend estimate** adapts to new information. Given that this holdout sits in the middle of a strong, sustained seasonal ramp (steadily climbing week over week into the winter peak), reason through: would you expect a *high* β (trend adapts fast, closely tracks the current climb) or a *low* β (trend estimate stays stable, slow to change) to perform better here — and does your grid search result match that reasoning? Write two or three sentences connecting the winning parameter back to what's actually happening in the data, the way Lecture 1's decomposition taught you to think about it.

### Part C — Do the same for single exponential smoothing (10 min)

Grid search SES's single parameter α over `[0.1, 0.2, ..., 0.9]` on the same holdout. Does *any* α let plain SES catch up to seasonal naive or your tuned Holt? If not, explain in one sentence what SES is structurally missing that no amount of α-tuning can fix (you answered this in Lecture 3 — restate it here in your own words, grounded in this specific result).

### Part D — The judgment call: would you actually ship the tuned Holt? (10 min)

This is the part with no clean answer. You tuned α and β to minimize error on **exactly the 12 weeks you're now bragging about beating seasonal naive on.** That's a legitimate concern: a model tuned to minimize error on a specific holdout can be *overfit to that holdout* — it may not perform as well on the *next* 12 weeks, which you haven't seen yet and can't tune against without cheating.

Answer, with reasoning:

1. Would you trust this tuned (α, β) pair to generalize to next year's ramp, or might it be overfit to this particular 12-week window's specific noise?
2. What would you do differently to get more confidence that a tuned Holt genuinely beats seasonal naive, rather than just beating it on one lucky window? (You don't need to implement this — describe the approach. Multi-window backtesting is coming properly in Week 4; a one-sentence preview of what you'd want is a strong answer here.)
3. Given the practical cost difference — seasonal naive requires zero tuning and one year of history; Holt requires a grid search and ongoing re-tuning as the business changes — would you recommend the tuned Holt for production, or seasonal naive, or something else entirely? Defend your pick in 3–4 sentences to a VP who does not want to hear the word "hyperparameter."

## Constraints

- Use the exact same 12-week holdout throughout (`2024-10-07` – `2024-12-23`) so your results are comparable to the lecture's.
- Show your grid search results (at least the top 5 rows by MAE), not just the winning number — a reader should be able to see how sensitive the result is to the parameter choice.
- Part D must take a position. "It depends" is a valid final sentence, but only after you've stated what it depends on and which way you'd lean given what you know now.

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Grid search | Tries 2–3 combinations by hand | Systematic grid, results table shown |
| Reasoning about β | States the winning value | Connects the winning value to the shape of the holdout (sustained ramp → needs fast trend adaptation) |
| SES comparison | Skipped or asserted without numbers | Actually grid-searched, actually shows SES can't close the gap |
| Overfitting awareness | Declares victory because MAE went down | Explicitly names the overfitting risk of tuning on the same window you're scoring |
| Recommendation | No clear pick | A defended, VP-readable recommendation that accounts for both accuracy and operational cost |

## Submission

Commit `challenge-02.md` (plus your grid-search code) to your portfolio under `c40-week-03/challenge-02/`.
