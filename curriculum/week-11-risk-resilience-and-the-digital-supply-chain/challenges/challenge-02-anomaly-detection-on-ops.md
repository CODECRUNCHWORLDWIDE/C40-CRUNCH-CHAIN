# Challenge 2 — Anomaly Detection on Ops Data

**Time:** ~90 minutes. **Difficulty:** Medium-hard. **This one has a genuine sweet spot to find.**

## The scenario

Exercise 3 built a working z-score pipeline at a fixed threshold (`z < -2`, persistence = 2 days). It works, but "z < -2" was asserted, not chosen. Your job this time is to actually **find the right threshold** by putting a dollar cost on both kinds of mistake a detector can make — flagging something that isn't real (false positive) and missing something that is (late or missed detection) — and picking the threshold that minimizes total cost.

## Given assumptions (the cost model)

- **False-positive cost: $150 per flagged day** on a DC that was **not** meaningfully disrupted (Memphis DC or Reno DC this week) — this represents an analyst's time spent investigating a flag that turns out to be nothing.
- **Delay cost: $1,800 per day** between the *most sensitive reasonable threshold's* confirmed-detection date and your chosen threshold's confirmed-detection date, for Austin East — this represents the growing cost of an unmanaged disruption per Exercise 2's finding that earlier action shrinks the damage.
- **Non-detection penalty: $50,000 flat**, if your threshold is so tight that the persistence rule (2 consecutive flagged days) never fires at all for Austin East within the 60-day window — this represents the disruption running completely undetected until it surfaces some other way (a customer complaint, a monthly report).

## Your task

1. **Build the sweepable pipeline.** Extend Exercise 3's pipeline into a function `evaluate_threshold(z_threshold)` that, for a given threshold, returns: the confirmed-detection date for Austin East (or `None`), and the total flagged-day count for Memphis DC and Reno DC combined (your false-positive proxy).

2. **Sweep it.** Run `evaluate_threshold` across at least eight thresholds spanning **z = -1.5 down to z = -5.0**. For each, compute `total_cost = (fp_days × $150) + (delay_days × $1,800) + (never_detected ? $50,000 : 0)`, where `delay_days` is measured against the confirmed-detection date at your most sensitive threshold (z = -1.5).

3. **Find the minimum.** Report the threshold with the lowest total cost, and show the full table (threshold, confirmed date, FP days, delay days, total cost) so the minimum is visible, not just asserted.

4. **Characterize the shape.** In 3-4 sentences: does cost fall smoothly to a minimum and then rise smoothly, or does it change more abruptly in one direction? What does that shape tell you about how forgiving (or unforgiving) this particular detection problem is to getting the threshold slightly wrong in each direction?

5. **Cross-check with IQR.** Independently of the z-score sweep, compute IQR-based outliers (Lecture 3, Section 2) for Austin East's `otif_pct` across the full 60-day series. List the flagged dates. Do they materially agree with your best z-score threshold's flagged dates? Name one structural reason IQR and rolling z-score could disagree even when both are "correct" by their own definition (hint: one looks at the whole series at once; the other updates day by day).

6. **Stretch — multivariate signal.** Build a composite anomaly score for Austin East by summing `|z|` across all three metrics (`otif_pct`, `fill_rate_pct`, `avg_lead_time_days`) computed the same rolling way. Does the composite score cross a "clearly anomalous" level any earlier than the `otif_pct`-only z-score did? Report the date it does.

## Constraints

- Use the given dollar assumptions exactly — don't substitute your own without flagging it.
- Your sweep must include enough thresholds to show the cost curve actually turning (rising again on one side), not just monotonically improving — if your sweep only goes from -1.5 to -3.0, you won't see the far side of the curve. Go far enough (down to at least -5.0) to see detection start to fail.
- Do the z-score and IQR calculations independently — don't derive one from the other.

## Hints

<details>
<summary>On why the cost curve has a genuine minimum</summary>

Two forces pull in opposite directions as the threshold tightens (more negative `z`): false-positive days on Memphis/Reno go **down** (fewer normal fluctuations cross an extreme threshold), which is good — but at some point the threshold gets so tight it also stops reliably catching *two consecutive* real anomalous days on Austin East, which either delays confirmation or prevents it entirely. Somewhere in between is a threshold tight enough to cut false positives hard but not so tight it endangers real detection. That's the minimum you're looking for — it should land somewhere in the neighborhood of `z` between -3 and -4, not at either extreme of your sweep.

</details>

<details>
<summary>On the IQR/z-score disagreement</summary>

IQR here is computed once over the whole 60-day series, so it "knows" about the disruption when judging *every* day, including the pre-disruption days — meaning its quartiles are pulled toward including the disruption's low values, which can make IQR's fences slightly wider (less sensitive) than a rolling z-score computed only from the pre-disruption baseline. This is exactly why a live control tower uses a rolling window, not a static IQR computed on the whole history to date — a static calculation gets contaminated by the very anomaly it's supposed to catch.

</details>

## How success is judged

| Signal | Weak submission | Strong submission |
|---|---|---|
| Sweep completeness | Only 2-3 thresholds tried, or doesn't reach far enough to see cost rise again | 8+ thresholds, full table shown, minimum clearly visible |
| Cost model correctness | Formula applied inconsistently or missing a term | All three cost components correctly computed at every threshold |
| IQR cross-check | Skipped, or z-score and IQR conflated into one calculation | Independently computed, dates compared, disagreement explained structurally |
| Reasoning | "Lower threshold is better" with no nuance | Explains *why* the minimum sits where it does, referencing both failure modes |

## Submission

Commit `challenge-02.py` (with your written findings as comments or a companion `challenge-02.md`) to your portfolio under `c40-week-11/challenge-02/`.
