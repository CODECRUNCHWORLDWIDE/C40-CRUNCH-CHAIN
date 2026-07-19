# Week 3 — Homework

Five problems, ~4.5 hours total, spread across the week. All against the `demand_history` seed from the [README](./README.md) unless a problem says otherwise. Commit each.

---

## Problem 1 — Decompose all six SKUs (60 min)

In Lecture 1 you decomposed a small toy example by hand. Now do it for real, in pandas, on all six core SKUs.

1. For each SKU, compute a centered moving-average trend using a 52-week window (`series.rolling(window=52, center=True).mean()` gets you close; a proper 2×52 centered technique is described in Lecture 1 Section 3 if you want to match it exactly).
2. Detrend each SKU (`actual − trend`), then compute an average seasonal index per ISO week.
3. For each SKU, report: the range of the seasonal index (max − min — a rough measure of "how seasonal is this SKU"), and the average residual magnitude (a rough measure of "how noisy is this SKU").

**Deliver** `decompose_all.py` and a short table (in comments or a printed DataFrame) ranking the six SKUs from "most seasonal" to "least seasonal," and separately from "noisiest" to "cleanest." Do the rankings match your intuition from reading Lecture 1's description of each SKU?

---

## Problem 2 — Twenty warm-up forecasts and scores (75 min)

Write and run each against `demand_history`. Put your SQL/pandas in `warmups.py` with a `# N` comment above each and the result printed beneath.

1. Naive forecast for `JCK-ALP-001`, last 8 weeks only.
2. Seasonal-naive forecast for `SAN-TRL-040`, last 8 weeks.
3. 4-week moving average for `BAG-DAY-020`, full series.
4. 8-week moving average for `BAG-DAY-020`, full series — compare its MAE on the last 12 weeks to the 4-week version's.
5. SES with α=0.5 for `FLC-ZIP-030`, last 12 weeks scored.
6. Holt with α=0.4, β=0.4 for `FLC-ZIP-030`, last 12 weeks scored — does it beat naive here? (Recall Challenge 1's discussion of this SKU.)
7. MAE of naive on `ACC-BEA-010`'s last 12 weeks.
8. RMSE of the same forecast from #7.
9. Bias of the same forecast from #7 — is it over- or under-forecasting, and by how much?
10. MAPE of seasonal naive on `JCK-STM-002`'s last 12 weeks, with the exclusion count reported.
11. Which of naive, seasonal naive, or MA4 has the lowest RMSE on `ACC-BEA-010`?
12. Total demand (`SUM(units_sold)`) for each SKU across all 104 weeks, in SQL, sorted descending.
13. The single highest-demand week across the whole dataset (any SKU) — which SKU, which week, how many units?
14. The single lowest non-zero demand week across the whole dataset.
15. How many weeks total (across all six SKUs) have `units_sold = 0`? Which SKU(s) do they belong to?
16. Average weekly demand for `JCK-ALP-001`, computed two ways: `AVG()` in SQL and `.mean()` in pandas. Confirm they match.
17. The 4-week moving average forecast for `JCK-STM-002` on `2023-11-20` (a promo week) — how far off was it from the actual? What does that tell you about MA4's ability to handle a one-off spike?
18. Seasonal naive's forecast for `JCK-STM-002` on `2024-11-18` (this year's Black Friday-adjacent week) — does it "predict" a spike, and why?
19. For `FLC-ZIP-030`, compute the average of the first 8 weeks vs. the average of the last 8 weeks of the series. What's the percentage growth, and does it match the "fastest-growing SKU" label from Lecture 1?
20. Pick any SKU and any method not otherwise required in this list, score it on the 12-week holdout, and state in one sentence whether you'd trust it in production.

---

## Problem 3 — Explain bias in your own words (45 min)

In `bias-writeup.md`, answer in prose (no more than 400 words total):

1. Compute the bias of naive, seasonal naive, and MA4 on `JCK-STM-002`'s 12-week holdout. All three should come out positive. Why — what's happening in the data that makes every simple method under-forecast this particular SKU?
2. Explain in your own words the operational difference between a forecast with high MAE and low |bias|, versus one with low MAE and high bias. Which failure mode is more dangerous for an inventory team, and why?
3. Give a real-world example (not from this course) of a business where consistent forecast bias (in either direction) would be more costly than random, unbiased error of the same average size.

---

## Problem 4 — MAPE across engines and edge cases (45 min)

The same computation, stress-tested against real edge cases. In `mape_edge_cases.py`:

1. Compute MAPE for `SAN-TRL-040`'s seasonal-naive forecast on the full 104-week series (not just the 12-week holdout). How many weeks get excluded for a zero actual?
2. Compute MAPE the "naive way" (without excluding zeros — let Python produce `inf` or a `ZeroDivisionError`) and observe what actually happens. Paste the error or the broken output.
3. Compute a MAPE variant that instead adds a small constant (e.g., 1) to every actual before dividing, to avoid the zero-division without dropping rows. Compare this number to your Task 1 result — are they close, or does the constant meaningfully distort the metric on a low-volume SKU?

**Deliver** the three computations plus a two-sentence recommendation: for a SKU with occasional zero-demand weeks, would you rather exclude those weeks from MAPE, patch them with a constant, or abandon MAPE for that SKU entirely in favor of MAE/RMSE?

---

## Problem 5 — Extend the dataset with a new SKU (60 min)

Make the data your own and forecast it.

1. Write a Python generator (reusing the pattern from `generate_50_skus.py` in the mini-project, or writing your own from scratch) for **one new SKU** of your invention — pick its category, level, trend, seasonal amplitude, peak week, and noise level deliberately, and write one sentence justifying each choice (e.g., "a rain jacket, so I gave it spring-peak seasonality").
2. Generate 104 weeks of demand for it and load it into `demand_history` and `sku_dim`.
3. Run all five forecasting methods against it, score them on the 12-week holdout, and report which one wins.
4. Then **undo** it cleanly: `DELETE FROM demand_history WHERE sku_id = 'YOUR-SKU-ID'; DELETE FROM sku_dim WHERE sku_id = 'YOUR-SKU-ID';` and confirm you're back to the original SKU count.

**Deliver** `new_sku.py` (the generator, the load, the scoring, and the cleanup) plus one sentence on whether the winning method matched what you expected given the parameters you chose.

---

## Time budget

| Problem | Time |
|--------:|----:|
| 1 | 60 min |
| 2 | 75 min |
| 3 | 45 min |
| 4 | 45 min |
| 5 | 60 min |
| **Total** | **~4.75 h** |

After homework, take the [quiz](./quiz.md) and ship the [mini-project](./mini-project/README.md).
