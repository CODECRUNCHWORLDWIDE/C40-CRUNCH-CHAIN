# Week 5 — Challenges

Two challenges, both harder and more open-ended than the exercises. Where the exercises had a clean expected number waiting at the end, these ask you to make and defend a judgment call — the exact skill an inventory or supply-chain analyst gets paid for once the formulas are second nature.

## How to work these

- Both challenges expect you to have finished all three exercises first — they reuse EOQ, safety-stock, and reorder-point logic without re-deriving it.
- Write your reasoning, not just your numbers. A correct `Q*` with no explanation of the trade-off it reflects is a half-answer here.
- Challenge 2 introduces its own small dataset (given in the challenge file) — you don't need any additional seed setup beyond what's already in the file.

## Files

| Challenge | Focus | Time |
|---|---|---|
| [challenge-01-newsvendor-optimal-order.md](./challenge-01-newsvendor-optimal-order.md) | Size a one-shot order under demand uncertainty, and show how the answer moves with the cost structure | 1h |
| [challenge-02-multi-echelon-inventory.md](./challenge-02-multi-echelon-inventory.md) | Centralize vs. decentralize safety stock across a 3-warehouse network; quantify risk pooling | 2h |

## Done when…

- [ ] You've solved a newsvendor problem from a cost structure, not a pre-given critical ratio, and can explain what changing the salvage value does to the answer.
- [ ] You've computed total safety stock two different ways for a multi-echelon network and can state, in dollars, the value of risk pooling.
