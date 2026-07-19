# Exercise 3 — Run a Rolling-Origin Backtest

**Goal:** Build a rolling-origin backtest loop by hand — no shortcut library call — so you understand exactly what "no leakage" means mechanically, then use it to compare three models honestly on the same folds.

**Estimated time:** 2 hours.

## Setup

Same SKU: `TNT-220`, Northeast region. Build the lag/rolling/calendar feature matrix from Lecture 3 §1 (reuse your code, don't rebuild it) and confirm `d2` (post-`dropna`) has 104 usable rows.

## Tasks

1. **Write the fold generator by hand.** Write a function `rolling_origin_folds(n, fold_size=13, n_folds=4, min_train=60)` that returns a list of `(train_idx, test_idx)` index-array pairs, walking backward from the end of the series, stopping early if a fold would leave fewer than `min_train` training rows. Test it on a plain `range(104)` first and print the `(train_end, test_start, test_end)` boundaries for each fold — verify by eye that no fold's test indices ever overlap another fold's, and that every train index is strictly less than every test index in its own fold. *(This is the exact check that catches an off-by-one leakage bug before it contaminates every number downstream.)*

2. **Score three models on the same folds**, using your fold generator:
   - **Seasonal naive** — `lag_52` as the prediction, no fitting required.
   - **4-week moving average** — `roll_mean_4` as the prediction, no fitting required.
   - **Gradient-boosted tree ensemble** — fit fresh inside each fold (never reuse a model fit on a different fold's training window), same hyperparameters as Lecture 3 §2.

   For each fold, record MAE for all three models in a small results table (`pandas.DataFrame` with columns `fold, model, mae`).

3. **Plot or print the per-fold MAE** for all three models side by side. Does one model win every single fold, or does the ranking flip between folds? State the answer in `notes.md` — this is exactly the "is the gap consistent" question from Lecture 3 §6.

4. **Deliberately introduce the leakage bug**, run it once, and see the damage: recompute `roll_mean_4` **without** the `.shift(1)` (i.e., let the rolling window include the current row), refit the tree ensemble with this leaked feature on **one** fold, and compare its MAE to the leakage-free version from Task 2. *(Expected: the leaked version's MAE will look noticeably better — that's the leak working exactly as badly as advertised. Delete this leaked feature and don't reuse it — this task exists purely so you've seen leakage's fingerprint with your own eyes, once, in a low-stakes exercise, before you might miss it in a real project.)*

5. **Write the honest verdict** in `notes.md`: which of the three models had the lowest mean MAE across folds, was the win consistent, and — per Lecture 3 §6 — is the gap between the winner and the seasonal-naive baseline big enough, relative to this SKU's average weekly volume, to justify the extra complexity of a tree ensemble in a real operations setting?

## Expected result (sanity range, not an exact target)

| Model | Mean MAE across folds (rough range) |
|---|---:|
| Seasonal naive | 18–23 |
| 4-week moving average | 14–19 |
| Gradient-boosted trees | 11–16 |

The tree ensemble should win on mean MAE, and it should win (or nearly tie) in most individual folds, not just on average — if your ranking flips wildly fold to fold, double-check each fold is refitting the model fresh (Task 2) rather than reusing one fit across folds, which itself is a subtler form of leakage (later folds' training data would be influencing predictions on earlier folds' test data via a stale model).

## Done when…

- [ ] `folds.py` contains your hand-written `rolling_origin_folds` function, tested and verified for no train/test overlap.
- [ ] `backtest.py` produces a `results.csv` with per-fold, per-model MAE for all three models.
- [ ] `notes.md` documents the leaked-vs-clean comparison from Task 4, with both MAE numbers shown.
- [ ] `notes.md` gives an honest verdict per Task 5 — a real recommendation, not just "the tree ensemble won."

## Stretch

- Extend the loop to also track **mean signed error** (bias), not just MAE, per fold per model. Does any model show a consistent directional bias (Lecture 3 §5's extrapolation problem)?
- Re-run the whole backtest with `fold_size=4` (monthly-ish, more folds, shorter horizon each) instead of `fold_size=13`. Does the ranking of models change at a shorter forecast horizon?

## Submission

Commit `folds.py`, `backtest.py`, `results.csv`, and `notes.md` to your portfolio under `c40-week-04/exercise-03/`.
