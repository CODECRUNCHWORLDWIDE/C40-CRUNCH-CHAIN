# Week 8 — Challenges

Two challenges, both harder and more open-ended than the exercises. Where the exercises had a clean expected number waiting at the end, these ask you to make and defend a judgment call against a real constraint the exercises deliberately simplified away — the exact skill a warehouse or fulfillment analyst gets paid for once the mechanics are second nature.

## How to work these

- Both challenges expect you to have finished all three exercises first — they reuse the ABC classification, re-slot, and batching logic without re-deriving it.
- Write your reasoning, not just your numbers. A correct staffing count or slotting plan with no explanation of the trade-off behind it is a half-answer here.
- Challenge 2 introduces its own dataset (given in the challenge file) — a peak-day scenario decoupled from Austin East DC's regular one-month pick profile, so you don't need any additional seed setup beyond what's already in the file.

## Files

| Challenge | Focus | Time |
|---|---|---|
| [challenge-01-slotting-optimization.md](./challenge-01-slotting-optimization.md) | Re-slot under a replenishment-frequency constraint, not distance alone | 1.5h |
| [challenge-02-labor-capacity-planning.md](./challenge-02-labor-capacity-planning.md) | Size a picking crew to a stated peak-day throughput target, with realistic slack | 1.5h |

## Done when…

- [ ] You've re-slotted the DC under a constraint that makes pure velocity-ranking insufficient, and can defend which SKU(s) you deliberately did **not** put in the golden zone despite high pick frequency.
- [ ] You've sized a picking crew for a stated peak-day volume, including a realistic utilization buffer, and can state what happens to order cycle time if the plan is understaffed by one picker.
