# Exercise 3 — Optimize & Report

**Time:** ~2 hours · **Tools:** Python (pandas, pulp)

Build and solve the network-flow LP from Lecture 2 §4, re-run the inventory policy against the **optimized** DC assignment, and assemble the full baseline-vs-optimized cost report from Lecture 2 §6. This is the exercise that turns three separate calculations into one number a stakeholder can act on.

## Task 1 — Build and solve the LP (45 min)

Implement Lecture 2 §4's `build_and_solve_network_lp()` exactly as written, using `region_demand` aggregated from Exercise 2's `demand_forecast` table (sum `forecast` across all 6 SKUs, grouped by `region_id`) rather than the hardcoded `DEMAND` dict in the lecture — this is the real integration point: your forecast, not a hand-typed number, drives the LP.

```python
region_demand = demand_forecast.groupby("region_id")["forecast"].sum().to_dict()
plan, outbound_cost, status = build_and_solve_network_lp(region_demand, CAPACITY, LANE_COST)
assert status == "Optimal"
```

Print `plan` and `outbound_cost`. Because your forecast comes from real seasonal-naive averages rather than the lecture's clean round numbers, expect `outbound_cost` to land **close to but not exactly** $40,550 — report your actual value and explain, in one sentence, why a small difference from the lecture's hand-worked number is expected here (hint: which month did you forecast, and does its seasonal index equal exactly 1.0?).

## Task 2 — Re-run inventory policy against the optimized plan (35 min)

Build a new `dc_assignment` dict from your solved `plan` — for each region, the DC that supplies the largest share of its demand (per Lecture 2 §5's `dc_assignment` line). Re-run `compute_inventory_policy()` from Exercise 2 with this **optimized** assignment, and compute `dc_level_holding_cost()` (Lecture 2 §3) using the optimized DC volumes from `plan`.

## Task 3 — Assemble the full report (40 min)

Combine everything into one `report.md` with:

1. A table matching Lecture 2 §6's format: baseline vs. optimized, for outbound freight, DC fixed cost, safety-stock holding cost, and the total.
2. The optimized routing `plan`, stated in plain English (which DC serves which region, and how the split works if any region is served by two DCs).
3. One sentence confirming every region's forecast demand was fully met by the plan (i.e., the LP's `status` was `"Optimal"`, not `"Infeasible"`).
4. The percentage cost reduction versus the $174,229/mo baseline, and whether it clears the 6% target from Lecture 1's scoping document.

## Expected outcome

| Component | Baseline | Your optimized result (approx.) |
|---|---:|---:|
| Outbound freight | $49,250 | ~$40,000–41,500 |
| DC fixed cost | $111,000 | $111,000 (unchanged — out of scope) |
| Safety-stock holding cost | $13,979 | ~$700–900 |
| **Total** | **$174,229** | **~$151,500–153,500** |
| **% reduction** | — | **~11.5–13%** |

Your exact numbers will differ slightly from Lecture 2's because your forecast is computed from a specific target month (`"2026-07"`, per Exercise 2) rather than the lecture's clean annual averages. A result **meaningfully outside** this range (say, a 25% reduction, or a 2% reduction) usually means a units mismatch — check whether `region_demand` accidentally used a single SKU's forecast instead of the summed total across all six.

## Deliverable

A directory `exercise-03/` containing `optimize.py`, `report.md`, and the solved `plan` saved as a small CSV (`dc_id,region_id,units`).

## Rules

- The LP's demand values must come from Exercise 2's `demand_forecast` table, not be re-typed from the lecture.
- `report.md`'s numbers must all be traceable to `optimize.py`'s output — no manually adjusted figures, even if they'd match the lecture more neatly.
