# Exercise 2 — Regression with Promo Features

**Goal:** Build a causal feature matrix, fit a regression, hit the same collinearity trap Lecture 2 hit — deliberately, so you recognize it fast in your own future work — diagnose it with VIF, and fix it.

**Estimated time:** 90 minutes.

## Setup

Same SKU as Exercise 1: `TNT-220`, Northeast region.

```python
import pandas as pd, numpy as np
df = pd.read_sql(
    "SELECT week_start, units, unit_price, promo_flag "
    "FROM weekly_demand WHERE sku = 'TNT-220' AND region = 'Northeast' ORDER BY week_start",
    con=engine,
)
```

## Tasks

1. **Build the feature matrix.** Add `t` (integer trend), `weekofyear`, `sin52`/`cos52` (cyclical calendar encoding), `log_price`, and `promo` (0/1). Reuse Lecture 2's code — don't re-derive the formulas from scratch, the point of this exercise is applying the workflow, not rediscovering trigonometry.

2. **Fit the full model** — `t`, `log_price`, `promo`, `sin52`, `cos52` — with OLS on all but the last 13 weeks. Print the coefficient table.

3. **Check the sign on `log_price`.** Is it negative (as expected for a normal good) or positive (a red flag, per Lecture 2 §2)? State which, in `notes.md`.

4. **Run the VIF diagnostic** (`variance_inflation_factor`, same code as Lecture 2 §3) on every feature. Report the VIF for `log_price` and `promo` specifically.

5. **Check the correlation directly**: `d["log_price"].corr(d["promo"])`. State the value in `notes.md` and explain in one sentence what it tells you about how promotions are run for this SKU (are promo weeks *always* price-cut weeks here too, or only sometimes?).

6. **Fix it.** Drop whichever of `log_price`/`promo` is redundant, re-fit, and confirm: (a) the sign on `log_price` is now correct, (b) the holdout MAPE is **unchanged** (or very close) versus the broken model — and explain in one sentence *why* dropping a perfectly redundant column doesn't cost predictive accuracy even though it fixes interpretability.

7. **State the semi-elasticity.** From the fixed model's `log_price` coefficient, compute "expected unit change per 1% price change" and express it as a percentage of this SKU's mean weekly demand, same calculation as Lecture 2 §3. Is this SKU price-elastic (>1 in magnitude) or inelastic (<1)?

## Expected result (sanity range, not an exact target)

- The broken model's `log_price` VIF should be extremely large or `inf` — if yours comes back under, say, 10, double-check you actually included `promo` as a raw 0/1 dummy (not accidentally dropped or mis-encoded).
- The correlation between `log_price` and `promo` should be strongly negative (close to −1) on this dataset — promotions here are built to always include a price cut.
- After the fix, holdout MAPE should land somewhere in the **5–9%** range, comparable to (not wildly different from) the broken model's MAPE.
- The corrected semi-elasticity should come out inelastic (magnitude below 1) for this SKU, similar in spirit to the jacket example in Lecture 2, though the exact number will differ.

## Done when…

- [ ] `notes.md` documents the broken sign, the VIF diagnostic, the correlation check, and the fix — in that order, mirroring how you'd actually debug this in a real job.
- [ ] The fixed model's coefficient table and holdout MAPE are both printed and saved.
- [ ] `notes.md` states the semi-elasticity and whether the SKU is elastic or inelastic, in plain language a merchandising manager (not a statistician) could act on.

## Stretch

- Add an interaction term: `promo_in_season` = `promo * (sin52 > 0.5)` (roughly, promos run near this SKU's summer peak). Does it improve holdout MAPE? Does its coefficient make business sense?
- Re-run the whole exercise on `JCK-100` in the **West** region instead of Northeast. Is the price/promo collinearity just as severe? (It should be — it's built into how promotions are structured in this dataset regardless of region — but confirm rather than assume.)

## Submission

Commit `regression.py` and `notes.md` to your portfolio under `c40-week-04/exercise-02/`.
