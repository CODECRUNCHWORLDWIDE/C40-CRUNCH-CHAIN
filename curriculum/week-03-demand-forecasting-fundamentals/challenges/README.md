# Week 3 — Challenges

Two open-ended problems. Unlike the exercises, these have **no single right answer** — the skill is judgment, backed by the numbers you compute. Do them after the three exercises.

1. **[Challenge 1 — Choose a baseline per SKU](challenge-01-choose-a-baseline-per-sku.md)** — for each of the six SKUs, pick a baseline method and defend the choice with your own scored numbers, not intuition. *(~60 min.)*
2. **[Challenge 2 — Seasonal naive vs. smoothing face-off](challenge-02-seasonal-naive-vs-smoothing.md)** — tune exponential smoothing against seasonal naive head-to-head, on a SKU where the lecture's ranking doesn't obviously hold, and argue for a production pick. *(~60 min.)*

## How these are judged

There's no answer key with exact numbers, because your computed numbers depend on the code you wrote in the exercises. Instead, each challenge tells you what a *strong* submission looks like. You're being graded on:

- **The score, not the vibe** — every recommendation must be backed by MAE/MAPE/RMSE/bias you actually computed, not "this one looks smoother."
- **Explicit trade-offs** — when two methods are close, say which metric you weighted more heavily and why, operationally.
- **Honesty about limitations** — if MAPE can't be computed cleanly for a SKU (hello, Trail Sandal), say so and use something else instead of hiding the problem.
- **A one-line takeaway a non-technical planner could act on** — "use seasonal naive for jackets, moving average for the Daypack" is worth more than a page of unread metrics.

Keep your work in `challenge-01.md` / `challenge-02.md` with your queries/code **and** your written reasoning. The reasoning is the point.
