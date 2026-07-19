# Challenge 2 — Argue Centralize vs. Distribute

**Time:** ~60 minutes. **Difficulty:** Medium-hard. **No single right answer.**

## The scenario

Crunch Gear's board is split. Half want to fund four new regional fulfillment centers next year to compete on delivery speed. Half think that's an expensive overreaction and want to stay centralized out of Memphis and invest the money in marketing instead. You've been asked to write the analysis memo that goes to the full board before the vote — not a recommendation with no reasoning shown, a memo the skeptical half can actually follow and argue with on the merits.

This is deliberately less scaffolded than Exercise 3. You won't be told every number — part of the exercise is stating what you assumed and why it's reasonable.

## What you're given

- Current state: 1 DC (Memphis), safety stock $500,000, fixed facility cost $420,000/year, average delivery time 4.5 days nationwide.
- Proposed state: 4 regional DCs, fixed facility cost **$175,000 each** ($700,000/year total), estimated average delivery time **1.8 days** for ~85% of customers (the remaining 15%, in low-density regions, stay closer to current speed).
- Inventory carrying cost rate: 22%/year (same as Exercise 3).
- Crunch Gear's DTC channel did **$18,000,000** in revenue last year, growing ~25%/year.
- A competitor offering 2-day delivery nationwide has been gaining share in Crunch Gear's category for the past two years.
- Customer research (internal survey, n=400): 62% of respondents said delivery speed "matters" or "matters a lot" when choosing between similar outdoor-apparel brands online.

## Your task

Write `challenge-02.md` as a **board memo** (headers, short paragraphs, at least one table) covering:

1. **The cost side.** Apply the square root law to estimate total safety stock under the 4-DC design, then compute and compare total facility + carrying cost for both designs (same method as Exercise 3, different numbers — show your work).

2. **The revenue-risk side.** You are **not given** a precise dollar figure for "revenue at risk from delivery speed." Make a stated, defensible assumption (e.g., "if even X% of the 62% of speed-sensitive customers are lost to the faster competitor per year without a response, that's $Y in lost DTC revenue") and show the resulting number. Be explicit that this is an estimate, and say what would sharpen it (a controlled pricing/delivery test, churn data, a win/loss analysis of lost deals).

3. **A genuine trade-off table.** Build a short table comparing the two designs on: annual cost, delivery speed, resilience to a single-site disruption (Lecture 3 §2), and capital/complexity to reverse the decision later if it doesn't pay off.

4. **Your recommendation, with the condition that would flip it.** State which design you'd recommend to the board, but also state explicitly: *"I would change this recommendation if [specific number or finding] turned out to be different."* This is the most important sentence in the memo — a memo with no stated condition for reversal reads as an opinion, not an analysis.

5. **One risk you'd flag either way.** Regardless of which design wins, name one operational risk the board should know about (e.g., WMS/systems readiness for 4 sites, hiring 4x the site-level management talent, inventory allocation logic needing to change) that isn't captured in the cost numbers at all.

## Constraints

- Show the square root law calculation — don't just assert a safety stock number.
- Every assumption you make that isn't given in the scenario must be labeled clearly (e.g., *"Assumption: ..."*) so the board can challenge it specifically instead of the whole memo.
- Keep it to roughly 500–700 words plus your table(s) — board memos that ramble don't get read.

## Hints

<details>
<summary>On estimating revenue at risk (task 2)</summary>

There's no single correct multiplier here. A reasonable structure: take the 62% "speed matters" figure, apply your own defensible discount for "matters" vs. "would actually switch brands over it" (that gap is real — stated preference in a survey overstates actual switching behavior), multiply by an assumed annual attrition rate to the faster competitor, and multiply by $18M. Whatever numbers you pick, the important part is that you *show* the chain of assumptions, not that you land on any particular final figure.

</details>

<details>
<summary>On the condition that would flip your recommendation (task 4)</summary>

Good conditions are specific and measurable: "if a controlled test showed DTC conversion improving by less than 1.5 points with faster delivery" or "if the safety stock estimate turns out to be 50% higher than assumed once real regional demand variance is measured." A vague condition ("if the market changes") doesn't count — it should be something the board could actually go and check.

</details>

## How success is judged

| Signal | Weak memo | Strong memo |
|---|---|---|
| Cost-side math | Missing or unlabeled | Square root law shown, totals computed and compared cleanly |
| Revenue-risk estimate | Ignored ("hard to say") or a bare unjustified number | A stated chain of assumptions leading to a number, with its own uncertainty acknowledged |
| Trade-off table | One-sided or missing a dimension | Covers cost, speed, resilience, and reversibility |
| Recommendation | A flat pick with no nuance | A pick **plus** a specific, checkable condition that would reverse it |
| Format | Wall of text | Reads like something a real board would actually get through |

## Submission

Commit `challenge-02.md` to your portfolio under `c40-week-01/challenge-02/`.
