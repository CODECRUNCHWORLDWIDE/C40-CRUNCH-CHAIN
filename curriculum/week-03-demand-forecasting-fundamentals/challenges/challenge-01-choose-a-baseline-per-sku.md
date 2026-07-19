# Challenge 1 — Choose a Baseline per SKU

**Time:** ~60 minutes. **Difficulty:** Medium. **No single right answer.**

## The scenario

You're the forecasting analyst at Crunch Gear. Your VP of Operations doesn't want a research paper — she wants a one-page recommendation: **"for each of these six SKUs, which baseline forecast do we put into production Monday morning, and why?"** She will act on your answer, so guessing wrong has a real cost: pick a method that lags a trend and you understock a fast-growing SKU; pick a method that over-trusts a shaky seasonal pattern and you overstock a SKU whose "seasonality" was actually a one-time promo.

You have exactly the tools from this week: naive, seasonal naive, moving average (any window you choose), and the four error metrics. No exponential smoothing yet — that's Challenge 2. This challenge is about **matching a simple method to a demand shape**, which is a skill you'll need even after you've learned fancier methods, because half of real forecasting work is knowing when the fancy method isn't worth it.

## Your task

For **each of the six SKUs** in `demand_history`:

1. **Score all three baseline methods** (naive, seasonal naive, moving average) on the same 12-week holdout style used in the lectures (last 12 weeks of the series) — MAE, MAPE (noting any excluded zero-actual weeks), RMSE, and bias for each.
2. **Recommend one method** for that SKU and justify it with the numbers — not just "lowest MAE," but a sentence connecting the number to *why*, referencing what you know about that SKU's trend/seasonality/noise from Lecture 1.
3. **Flag anything the numbers can't tell you** — e.g., a promo-contaminated peak, a SKU with too little history for seasonal naive to even run, a SKU where MAPE is unreliable.

Put all six recommendations in `challenge-01.md`, one section per SKU.

## The six SKUs

1. **`JCK-ALP-001` (Alpine Shell Jacket).** You already have the lecture's worked numbers for this one — use them as your template, don't just copy the recommendation without re-deriving it from your own exercise code.
2. **`JCK-STM-002` (Summit Down Jacket).** This one has two deliberate promo spikes baked into its history (see Lecture 1). Does seasonal naive's strength turn into a weakness here? What happens to your recommendation if you know a promo is planned again this year vs. not?
3. **`ACC-BEA-010` (Merino Beanie).** Low volume, mild seasonality, low noise. Is this SKU "easy" — do multiple methods land close together? If so, would you pick the cheapest one to compute instead of the marginally most accurate one, and is that a legitimate business reason?
4. **`BAG-DAY-020` (Daypack 22L).** No real seasonality. You already saw in Exercise 3 that seasonal naive underperforms here — quantify by how much, and recommend accordingly.
5. **`FLC-ZIP-030` (Fleece Half-Zip).** Strong upward trend, weak seasonality. Which baseline handles trend best, and which handles it worst? Is any baseline in this week's toolkit actually *good* here, or are you choosing "least bad"? (If it's the latter, say so — that's a legitimate, useful answer, and sets up why Lecture 3's trend-aware Holt method exists.)
6. **`SAN-TRL-040` (Trail Sandal).** Summer seasonality, declining trend, weeks that hit zero. Walk through what happens to each method — and to MAPE specifically — during the zero-demand winter weeks. What would you actually put into production for this SKU's low season, given the tools you have this week?

## Constraints

- Every recommendation needs a number attached to it, computed by your own code from Exercises 2–3 (or code you write fresh for this challenge) — no recommendation by eyeballing a chart.
- If MAPE can't be honestly reported for a SKU (a zero-actual week), don't report it uncritically — either exclude and say how many rows, or lead with MAE/RMSE instead and say why.
- Where two methods are close (within ~10% of each other on MAE), don't force a tiebreak on accuracy alone — name a *second* factor (cost to compute, how much history it requires, robustness to a data gap) that would break the tie in a real job.

## Hints

<details>
<summary>On the Summit Down Jacket's promo spikes (SKU 2)</summary>

Seasonal naive assumes "this year's same week looks like last year's same week." If last year's Black Friday week had a promo and this year's does too, seasonal naive accidentally gets the promo effect "for free" — it's not smart, it's lucky. If the promo moves, or doesn't repeat, seasonal naive will be confidently wrong in the same week it was previously right. A strong answer separates "this method scored well" from "this method scored well *for a reason I can defend continuing to rely on*."

</details>

<details>
<summary>On the Trail Sandal's zero weeks (SKU 6)</summary>

There is no clean trick here — that's the point. A forecast of exactly 0 for a SKU that occasionally, but not always, sells 0 in a given week is a real operational decision (do you zero out safety stock entirely, or keep a small buffer?). This challenge doesn't require you to solve inventory policy (that's Week 5) — it requires you to *notice* that the forecasting method's output has a real downstream consequence and say so.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Evidence | "MA looks smoother" | A scored table (MAE/MAPE/RMSE/bias) for all three methods, all six SKUs |
| Reasoning | Picks lowest MAE and stops | Connects the winning method to the SKU's trend/seasonality/noise profile from Lecture 1 |
| Skepticism | Reports every metric at face value | Flags the promo contamination (SKU 2) and the MAPE zero-division (SKU 6) without being told to |
| Practicality | Six different exotic recommendations | Willing to say "these three are close enough — pick the cheapest" when that's true |
| Communication | A wall of numbers | A one-sentence, VP-readable takeaway per SKU |

## Submission

Commit `challenge-01.md` (plus any supporting `.py`) to your portfolio under `c40-week-03/challenge-01/`.
