# Challenge 2 — Executive Recommendation Deck

**Time:** ~1.5 hours · **Tools:** Markdown (no code — this challenge is entirely about communication)

Turn Challenge 1's results into a one-page memo following Lecture 3's five-part structure, then answer three follow-up questions a real VP would ask — in writing, each grounded in a specific number, not a hand-wave.

## Part A — The memo (45 min)

Write `memo.md` using Lecture 3 §2's five-part structure (Headline, The ask, The evidence, The trade-off, Next steps and risks). Base it on your **Exercise 3** baseline-vs-optimized numbers (the steady-state recommendation), not Challenge 1's shock/capacity scenarios — those are Q&A material, not the headline.

Requirements:
- The headline sentence must contain exactly one number (the total monthly or annual savings) and no jargon a non-analyst wouldn't understand on first read.
- The evidence section may include **at most one table and one chart** — if you have more numbers you want to include, they belong in an appendix, not the body.
- The trade-off section must name a real cost of the recommendation (Memphis's capacity utilization jumping from near-zero to 100%, per Lecture 3's worked example, or a different genuine trade-off if your numbers differ) — "no downside" is not an acceptable trade-off section.

## Part B — Defend it (45 min)

In `qa.md`, answer these three questions as if you were in the room and had to respond immediately, in 3–5 sentences each, citing a specific number from your own pipeline output (not the lecture's example numbers, unless yours happen to match):

1. **"What happens if demand comes in 20% higher than forecast for the Northeast, like sales is telling us might happen next quarter?"** — answer using your Challenge 1 Part B results, not a guess.
2. **"We're being told Memphis's lease might force a capacity cut to 17,000 units. Does that change your recommendation?"** — answer using your Challenge 1 Part C results, including the shadow-price finding.
3. **"Why should I trust the safety-stock numbers? A 94% cut in safety stock sounds aggressive."** — answer by pointing to the backtested MAPE from Exercise 2, and explain in plain language (no formulas) why a flat day-count buffer and a demand-volatility-based buffer can produce such different numbers, in terms a manager without a stats background would accept.

## What "strong" looks like

- The memo is genuinely one page — if you print it, it fits. Padding it out to look more thorough is a common and easy-to-spot mistake; a VP reads length as a signal of how well you understand your own finding, not how hard you worked.
- Every number in `qa.md` traces to a specific run of your pipeline from Challenge 1 or Exercise 3 — grading will ask you to point to exactly which script produced it.
- Question 3's answer avoids jargon entirely — no "z-score," no "cycle service level" — while still being technically correct. If you can't explain `√LT` without the square root symbol, you don't yet understand why it matters; go back to Lecture 2 §3 before writing the answer.

## Deliverable

`challenge-02/memo.md` and `challenge-02/qa.md`.
