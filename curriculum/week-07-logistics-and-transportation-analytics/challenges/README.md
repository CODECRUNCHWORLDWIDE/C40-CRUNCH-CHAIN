# Week 7 — Challenges

Two open-ended problems. Unlike the exercises, these reward judgment and correct implementation over a single right answer — do them after all three exercises.

1. **[Challenge 1 — Savings-Algorithm Routing](challenge-01-savings-algorithm-routing.md)** — implement the Clarke-Wright savings algorithm from scratch and prove it beats nearest-neighbor on the Austin East delivery network. *(~90 min.)*
2. **[Challenge 2 — Mode-Selection Trade-off](challenge-02-mode-selection-tradeoff.md)** — a slate of upcoming shipments, each with a real deadline; choose the mode for each and defend the total cost against two naive baselines. *(~90 min.)*

## How these are judged

Neither challenge has a single numeric answer key the way the exercises do. Instead, each one tells you what a *strong* submission looks like. You're being graded on:

- **Correctness of implementation** — for Challenge 1, does your savings algorithm actually respect capacity and produce valid routes (every stop visited exactly once, no route over 120 cases)?
- **Quantified comparison** — "the savings algorithm did better" is not a finding; "the savings algorithm found a 322-mile solution vs. nearest-neighbor's 346 miles, a 6.9% improvement" is.
- **Stated reasoning** — for Challenge 2, every mode choice needs one sentence of justification tied to the cost/speed/reliability trade-off from Lecture 3, not just a number.

Keep your work in `challenge-01.py` (or `.sql`+`.py`) and `challenge-02.md`/`.sql` with your code **and** your written reasoning. The reasoning is the point.
