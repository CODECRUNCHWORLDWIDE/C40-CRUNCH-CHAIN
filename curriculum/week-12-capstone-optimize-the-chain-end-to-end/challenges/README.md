# Week 12 — Challenges

Two open-ended problems, done after all three exercises. Challenge 1 stress-tests the pipeline you just built; Challenge 2 forces you to defend its output the way a real stakeholder would push back on it. Neither has a single numeric answer key — both tell you what a strong submission looks like.

1. **[Challenge 1 — End-to-End Optimization Run](challenge-01-end-to-end-optimization.md)** — chain your three exercises into one script, run it across multiple months, and sensitivity-test the result against a demand shock and a capacity cut. *(~2.5h)*
2. **[Challenge 2 — Executive Recommendation Deck](challenge-02-executive-recommendation.md)** — turn Challenge 1's results into a one-page memo, then answer three tough follow-up questions in writing, the way a VP actually would ask them. *(~1.5h)*

## How these are judged

- **Correctness of the pipeline** — for Challenge 1, does it actually re-run cleanly end to end with no manual steps, and does it correctly report `"Infeasible"` rather than silently producing a wrong number when a scenario genuinely can't be met?
- **Quantified, not qualitative, findings** — "the network handled the demand shock fine" is not a finding; "at +20% demand, Memphis hits 100% capacity and $1,840/mo of demand must reroute to Austin at $0.31/unit higher cost" is.
- **A memo that survives pushback** — for Challenge 2, every answer to a follow-up question needs to be grounded in a specific number from your pipeline, not a hand-wave.

Keep your work in `challenge-01/` (script + results) and `challenge-02.md` (memo + Q&A). The written reasoning is worth as much as the code.
