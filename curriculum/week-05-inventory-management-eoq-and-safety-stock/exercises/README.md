# Week 5 — Exercises

Three exercises, done in order. Each builds directly on the last: Exercise 1 sizes *how much* to order, Exercise 2 sizes the *buffer* around uncertainty, Exercise 3 puts both together into a running policy and checks, empirically, that the policy actually delivers the service level you designed it for.

## How to work these

- Use the `skus` table seeded in the [week README](../README.md). Confirm it first: `SELECT COUNT(*) FROM skus;` should return `10`.
- Do the SQL version and the pandas version of each task where both are asked for — they should agree to rounding. Disagreement almost always means an arithmetic slip in one of them; use the mismatch to find your bug, don't just trust whichever number looks nicer.
- Keep a running script or notebook per exercise (`exercise-01.sql` / `exercise-01.py`, etc.) — you'll reuse this exact logic, barely modified, in the mini-project.
- Show your work for anything that isn't a single query — a two-line comment above a number ("used z=1.645 for 95% CSL") saves you (and a reviewer) from re-deriving your reasoning later.

## Files

| Exercise | Focus | Time |
|---|---|---|
| [exercise-01-compute-eoq.md](./exercise-01-compute-eoq.md) | EOQ, order frequency, and total cost for the full catalog | 1h |
| [exercise-02-safety-stock-from-service-level.md](./exercise-02-safety-stock-from-service-level.md) | Safety stock and reorder point at 90/95/99% cycle-service levels | 1.5h |
| [exercise-03-simulate-a-reorder-point.md](./exercise-03-simulate-a-reorder-point.md) | Simulate a year of demand under an (s,Q) policy; measure the realized service level | 1.5h |

## Done when…

- [ ] You have EOQ, order frequency, cycle time, and `TC(Q*)` for all 10 SKUs.
- [ ] You have safety stock and ROP at three service levels for all 10 SKUs, and can point to which SKU is most sensitive to raising the target.
- [ ] You've run at least one full-year simulation of a reorder policy and compared the realized service level to the design target.
