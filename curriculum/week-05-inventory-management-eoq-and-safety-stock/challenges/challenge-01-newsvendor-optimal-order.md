# Challenge 1 — Newsvendor Optimal Order

**Time:** ~60 minutes. **Difficulty:** Medium.

## The scenario

Crunch Gear is running a one-time pop-up: **Trailfest 2026**, a single weekend event. Marketing commissioned an event-exclusive **beanie + scarf bundle** that will never be produced again — whatever doesn't sell during Trailfest goes to end-of-season clearance afterward at a steep discount. Production has to place **one order**, weeks before the event, with no ability to reorder mid-event. This is a textbook newsvendor problem: order too few bundles and you turn away paying customers at your own event; order too many and you eat the clearance loss on every leftover unit.

## The numbers

| Input | Value |
|---|---|
| Bundle selling price | $45.00 |
| Bundle unit cost | $19.00 |
| Post-event clearance (salvage) value | $8.00 |

Marketing pulled attendance-adjusted bundle sales from **8 comparable past pop-up events**, all similar in size and audience to Trailfest 2026:

```
event_sales = [420, 505, 380, 610, 455, 390, 530, 470]
```

## Your task

### Part A — the base case

1. Compute the underage cost `Cu` and overage cost `Co` from the price/cost/salvage numbers.
2. Compute the critical ratio `CR = Cu / (Cu + Co)`.
3. Treat the 8 historical events as a sample from a normal demand distribution. Compute the **sample mean** and **sample standard deviation** (use `n-1` in the denominator — this is a small sample, not a full population).
4. Compute `z = Φ⁻¹(CR)` (use `scipy.stats.norm.ppf(CR)`) and the optimal order quantity `Q* = μ + z*σ`.
5. State `Q*` as a whole number of bundles, and write one sentence explaining, in plain terms, why it sits above or below the sample mean.

### Part B — sensitivity to the salvage value

6. Recompute `Q*` if the clearance salvage value were only **$2.00** instead of $8.00 (a worse liquidation channel — maybe there's no clearance partner this time, just a deep online markdown). Does `Q*` go up or down? By how much?
7. Recompute `Q*` if Crunch Gear negotiated a **guaranteed buy-back** from a wholesale liquidator at **$14.00** per unsold bundle. Does `Q*` go up or down this time?
8. In one or two sentences, explain the general pattern: what does a better salvage channel do to the optimal order quantity, and why does that make business sense even before you compute a single number?

### Part C — don't trust the normal assumption blindly

9. The normal-distribution shortcut (`Q* = μ + z*σ`) is convenient, but you only have 8 data points — is a normal distribution really justified, or is that an assumption of convenience? Compute `Q*` a **second way**: take the `CR`-th **empirical quantile** of the 8 raw historical values directly (pandas: `pd.Series(event_sales).quantile(CR)`, using linear interpolation between the two nearest observed values), with no normal-distribution assumption at all.
10. Compare your two `Q*` estimates (normal-model vs. empirical-quantile). Are they close? Which would you trust more with only 8 data points, and what would make you more confident in either one (more historical events? a different distributional assumption? both)?

## Constraints

- Do the base case both by hand (showing every intermediate number) and in a short Python script using `scipy.stats.norm.ppf`.
- Round `Q*` to a whole number of bundles in your final recommendation — you can't produce half a bundle.
- Every sensitivity result (Part B) needs one sentence of business interpretation, not just the new number.

## Hints

<details>
<summary>On why the empirical and normal-model answers might differ (Part C)</summary>

With only 8 observations, the sample standard deviation is itself a noisy estimate — a different 8 events could easily give you a meaningfully different `σ`, which moves `Q*` around even though the underlying "true" demand distribution didn't change. The empirical quantile sidesteps the normality assumption entirely but has the opposite problem: with only 8 points, the 70th percentile is being estimated from a very thin slice of data, interpolated between just two or three of your observed values. Neither method is free of uncertainty — a strong answer says so explicitly rather than presenting one number as the truth.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|---------------|
| Mechanics | `Cu`, `Co`, `CR`, `z`, `Q*` computed correctly | Same, plus every intermediate number is shown, not just the final answer |
| Sensitivity reasoning | Reports new numbers with no explanation | Explains *why* a better salvage channel raises `Q*`, in plain business language |
| Model humility | Treats the normal-model `Q*` as the single correct answer | Computes the empirical-quantile alternative and reflects honestly on which to trust and why |
| Communication | Numbers only | A one-paragraph recommendation a Trailfest event manager could act on without a stats background |

## Submission

Commit `challenge-01.md` (write-up) and `challenge-01.py` (code) to your portfolio under `c40-week-05/challenge-01/`.
