# Challenge 2 — Hierarchical Forecast Reconciliation

**Time:** ~90 minutes. **Difficulty:** Hard.

## The scenario

`weekly_demand` has three SKUs across two regions — six **leaf** series (`SKU x region`), which sum up to a **grand total** (all SKUs, all regions). If you forecast each leaf series independently and someone else forecasts the grand-total series independently, there is no mathematical reason those two efforts agree — and on real data, they almost never do. This is called **incoherence**, and it's not a hypothetical: it's what happens by default any time forecasts are built at more than one level of a hierarchy without deliberately reconciling them.

Fit independent, damped-trend, additive-seasonal Holt-Winters models on all six leaves and separately on the grand total, and you get exactly this problem:

```python
bottom_up_total  = sum of the 6 independently-fit leaf forecasts
direct_total_fc  = an independent Holt-Winters fit on the grand-total series itself
```

On this course's dataset, over a 13-week holdout, these two numbers **differ by as much as ~14 units in a single week** even though both are reasonable, honestly-fit forecasts of the exact same thing (total units across the company). Imagine explaining to a VP why "the sum of what we're planning to stock across all SKUs" doesn't match "the total we're forecasting to sell" — that credibility problem is what reconciliation exists to prevent.

## Two standard fixes, and their real tradeoff

**Bottom-up**: forecast every leaf independently, then get the total (and any other aggregate level) purely by summing. Coherent by construction — sums always add up because the total *is* a sum, never an independent estimate. On this dataset:

| | Total-level 13-wk MAPE |
|---|---:|
| Bottom-up (sum of 6 independent leaf forecasts) | 4.70% |
| Direct Holt-Winters on the grand total | 4.81% |

Bottom-up slightly **wins** at the total level here — each leaf's own price/promo/seasonal dynamics turn out to be informative enough that summing them beats fitting the blended total directly.

**Top-down**: forecast only the total, then split it down to each leaf using a fixed historical proportion (leaf's share of total volume over the training window). Also coherent by construction, for the opposite reason — the leaves are defined *as* fractions of the total, so they can never fail to sum back to it. But look at what happens at the **leaf** level for `JCK-100`/Northeast specifically:

| | Leaf-level 13-wk MAPE (`JCK-100`, Northeast) |
|---|---:|
| Bottom-up (leaf's own independent forecast) | 12.33% |
| Top-down (grand-total forecast x historical share, 37.9%) | 25.39% |

Top-down is **more than twice as wrong** at the leaf level. The mechanism: `JCK-100` peaks hard in winter, `TNT-220` peaks hard in summer — their seasonal swings partly cancel when summed into the grand total, so the *total's* seasonal amplitude is much flatter than `JCK-100`'s own. Multiplying a flat total forecast by a fixed proportion completely misses `JCK-100`'s real, sharp winter peak. Top-down assumes every leaf shares the aggregate's seasonal shape, scaled by a constant — an assumption that's badly wrong whenever leaves have genuinely different seasonal timing, exactly the case here.

## Your task

1. **Reproduce both numbers above** — bottom-up and direct-total MAPE at the total level, and bottom-up vs. top-down MAPE at the `JCK-100`/Northeast leaf level. Confirm your own fit lands in the same ballpark (exact numbers may shift slightly with `statsmodels` version or convergence, but the *pattern* — top-down badly underperforming at the leaf level — should reproduce).

2. **Explain the mechanism in your own words**, in `notes.md`: why does top-down do fine at the total but badly at a leaf with a seasonal shape that diverges from the aggregate's? Sketch (in words or a small plot) `JCK-100`'s seasonal shape vs. `TNT-220`'s vs. the grand total's, and point at where they diverge most.

3. **Extend the hierarchy one more level.** This dataset actually supports a 3-level hierarchy: `SKU x region` (6 leaves) → `SKU total` (3, summed across regions) → `region total` (2, summed across SKUs) → `grand total` (1). Pick **either** the SKU-total or region-total middle level and compute bottom-up forecasts for it two different ways: (a) sum the 6 leaf forecasts appropriately, and (b) fit an independent Holt-Winters model directly on that middle-level series. Do they diverge as much as the leaf-vs-top comparison did? State a hypothesis for why the divergence would be bigger or smaller at this middle level compared to the full leaf-to-top comparison above.

4. **Propose (don't necessarily implement) a better reconciliation.** Both bottom-up and top-down are simple heuristics; a real forecasting platform typically uses a proper reconciliation algorithm — **MinT (trace minimization)** is the standard modern approach, which takes independent forecasts at *every* level and finds the coherent set of numbers closest to all of them, weighted by each series' forecast error variance, rather than picking one level as "ground truth" and deriving the rest. Read enough about MinT (see [`resources.md`](../resources.md)) to explain in 3–4 sentences, in your own words, what problem it solves that neither bottom-up nor top-down solves on their own. You do not need to implement MinT for this challenge — explaining what it does and why it would help here is the deliverable.

5. **Make a recommendation.** For Crunch Gear specifically — a business whose SKUs have genuinely different seasonal shapes (winter jackets, summer tents) — would you default to bottom-up, top-down, or flag this as a case where the extra complexity of MinT-style reconciliation is worth it? Defend your answer with the numbers from Task 1.

## Deliverable

`notes.md` covering all five tasks, plus `reconciliation.py` with the code that reproduces the MAPE table above (and your Task 3 middle-level extension).

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|-------:|---------------------|
| Reproduces the core comparison | 25% | Bottom-up/direct-total and bottom-up/top-down leaf comparison both reproduced, same pattern as reported above |
| Mechanism explanation | 25% | Correctly identifies seasonal-shape divergence between leaves as the cause of top-down's leaf-level failure |
| Middle-level extension | 20% | Genuinely computes both ways at the chosen middle level, states a defensible hypothesis about the divergence size |
| MinT explanation | 15% | Correctly describes MinT as reconciling *all* levels' independent forecasts toward mutual consistency, not "pick one level and derive the rest" |
| Recommendation | 15% | Specific to Crunch Gear's actual seasonal structure, backed by the numbers, not generic advice |

## Submission

Commit `reconciliation.py` and `notes.md` to your portfolio under `c40-week-04/challenge-02/`.
