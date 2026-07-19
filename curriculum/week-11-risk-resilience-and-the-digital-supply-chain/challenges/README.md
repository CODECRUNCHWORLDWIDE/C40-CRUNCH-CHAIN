# Week 11 — Challenges

Two open-ended problems. Unlike the exercises, these reward judgment and a correctly-argued recommendation over a single right answer — do them after all three exercises.

1. **[Challenge 1 — Resilience Network Redesign](challenge-01-resilience-network-redesign.md)** — cost two resilience plans against Crunch Gear's top two unmitigated risks and recommend which one is actually worth building. *(~90 min.)*
2. **[Challenge 2 — Anomaly Detection on Ops Data](challenge-02-anomaly-detection-on-ops.md)** — tune a real anomaly detector across all three DCs; measure detection lag and the false-positive cost of getting the threshold wrong. *(~90 min.)*

## How these are judged

Neither challenge has a single numeric answer key the way the exercises do. Instead, each one tells you what a *strong* submission looks like. You're being graded on:

- **Correctness of the model** — for Challenge 1, does your expected-annual-loss math actually follow the `likelihood × impact` framework from Lecture 1, with every input traced back to a stated assumption or an earlier exercise's measured number?
- **Quantified comparison** — "the buffer helps" is not a finding; "the buffer cuts expected annual loss by $47,125 at a carrying cost of $27,300, a net benefit of $19,825/year" is.
- **Stated reasoning** — every recommendation needs the "why," tied to the specific numbers you computed, not a general appeal to "resilience is good."

Keep your work in `challenge-01.md` (or `.sql`/`.py` alongside it) and `challenge-02.py` with your code **and** your written reasoning. The reasoning is the point.
