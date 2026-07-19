# Week 9 Challenges — Overview

Two challenges. Both stretch past the exact patterns in the lectures — you'll need to extend a formulation, not just re-run one. Neither has a single provided "correct number" to check against; instead, each gives you a way to sanity-check that your model is behaving correctly. That's deliberate: real optimization work rarely comes with an answer key, only with checks for internal consistency.

| # | Challenge | Extends | Time |
|--:|-----------|---------|-----:|
| 1 | [challenge-01-multi-product-network-design.md](./challenge-01-multi-product-network-design.md) | Lecture 2's transportation model → two products sharing plant capacity | 1.5h |
| 2 | [challenge-02-capacitated-production-plan.md](./challenge-02-capacitated-production-plan.md) | Lecture 1's production-mix model → multiple time periods with inventory | 1.5h |

## How to work them

1. **Re-read the relevant lecture section before starting** — both challenges are direct extensions of a pattern you've already built once.
2. **Extend the index sets first.** Both challenges add a new dimension (a product, a time period) to a model you already know how to write for one dimension. The trick is almost always in how you index your variables — `x[plant, dc]` becomes `x[plant, dc, product]` or `x[product, period]` — not in the underlying LP logic.
3. **Validate with a bound or a decomposition check**, not a memorized number. Each challenge tells you what sanity check to run.
4. **Write down your assumptions.** Where the challenge leaves something ambiguous (a starting inventory, a rounding rule), state your choice explicitly in your solution file.
