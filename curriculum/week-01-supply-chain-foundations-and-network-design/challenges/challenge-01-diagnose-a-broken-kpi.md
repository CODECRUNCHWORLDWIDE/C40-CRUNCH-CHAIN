# Challenge 1 — Diagnose a Broken KPI

**Time:** ~60 minutes. **Difficulty:** Medium. **No single right answer.**

## The scenario

You're the newest analyst on Crunch Gear's operations team. The VP of Operations forwards you last quarter's dashboard, visibly relieved:

> "Good news — fill rate is 96.4% and OTIF is holding at 90%. But customer complaints to support are up 40% quarter-over-quarter, and our biggest wholesale account just emailed asking whether they should 'reconsider the relationship.' I don't get it — the numbers look fine. Can you figure out what's actually going on before our call with them Friday?"

Something doesn't add up: the headline metrics look healthy, but the people closest to the customer relationship are alarmed. Your job is to find the gap between what the dashboard shows and what the customer is actually experiencing — and explain it in terms the VP (who does not want a statistics lecture) will immediately understand.

## What you're given

The VP's dashboard, as reported:

| Metric | This quarter | Last quarter |
|---|---:|---:|
| Unit fill rate | 96.4% | 95.8% |
| OTIF | 90.0% | 91.0% |
| Support complaints | +40% QoQ | — |
| Damage claims (freight) | 6.5% of shipments | 3.0% of shipments |
| Invoice discrepancies reported by customers | 4.0% of orders | 2.5% of orders |

Notice: the dashboard the VP is looking at **does not include a perfect order rate** — it was never built as a tracked metric. Damage claims and invoice discrepancies are tracked separately, by different teams, and nobody had connected them to the "is the customer happy" question before now.

## Your task

In `challenge-01.md`:

1. **Compute the real picture.** Using this quarter's numbers, estimate a **composite perfect order rate**, treating OTIF, damage-free rate, and invoice-accuracy rate as (roughly, for this estimate) independent components that must all succeed for an order to be "perfect," per Lecture 2 §3. Show the multiplication.

2. **Compare it to what the VP is seeing.** Write 3–4 sentences translating the gap between "96.4% fill rate, 90% OTIF" (what's on the dashboard) and your computed perfect-order estimate (what's actually happening to a customer, order to order). Use plain language — assume the VP has never heard the phrase "compounding failure rate."

3. **Explain the trend, not just the level.** Damage claims *doubled* quarter over quarter (3.0% → 6.5%) and invoice discrepancies grew too, while fill rate and OTIF barely moved. In 2–3 sentences, explain why a dashboard built only around fill rate and OTIF would completely miss this — and why the wholesale account's complaint makes total sense once you look at the components that *did* move.

4. **Recommend what changes on the dashboard.** What should Crunch Gear track going forward so this gap doesn't hide again? Be specific — name the metric(s), not just "track more things."

5. **One sentence for Friday's call.** Write the one sentence the VP should say to the wholesale account that acknowledges the real problem, without either over-promising or drowning them in KPI jargon.

## Constraints

- Don't just say "perfect order was low" — show the actual multiplication and the resulting number.
- Assume the independence approximation from Lecture 2 (component rates multiply) even though you know real-world failures aren't perfectly independent — that's a reasonable estimation tool here, not a claim of statistical rigor. You may note this caveat in one sentence if you want.
- Your recommendation in task 4 must be something a real ops team could implement, not a vague call for "better quality."

## Hints

<details>
<summary>On the composite calculation (task 1)</summary>

You have three component rates to combine: OTIF (90.0%), damage-*free* rate (100% − 6.5% = 93.5%), and invoice-*accuracy* rate (100% − 4.0% = 96.0%). Multiply all three together, the same way Lecture 2 §3 multiplied four 95% components down to ~81%.

</details>

<details>
<summary>On why the trend matters more than the level (task 3)</summary>

OTIF and fill rate are the two metrics that were *already on the dashboard*, so the VP was watching them and saw them barely move — which is exactly why the VP is confused. The two metrics that actually deteriorated (damage, invoice accuracy) were being tracked by other teams and never rolled into "is the customer experience OK?" A KPI that isn't on the shared dashboard might as well not exist for decision-making purposes, even if someone, somewhere, has the number in a spreadsheet.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|---|---|---|
| Composite math | Skipped or estimated without showing work | Multiplication shown, correct, and clearly labeled |
| Plain-language translation | Reuses jargon like "compounding" without unpacking it | Explains the gap the way you'd explain it to a non-technical VP |
| Root-cause reasoning | Blames "operations" vaguely | Points specifically at the damage/invoice trend as the actual driver |
| Actionable recommendation | "Track quality better" | Names specific metric(s) and where in the process they'd be measured |

## Submission

Commit `challenge-01.md` to your portfolio under `c40-week-01/challenge-01/`.
