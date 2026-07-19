# Exercise 2 — Compare Aggregate-Plan Strategies

**Goal:** apply Lecture 2's chase/level/mixed method to Trail Footwear's **real** demand numbers and cost structure, in Python, and confirm the same pattern (level cheapest, chase most expensive) holds — or doesn't — once the numbers get harder.

**Estimated time:** 90 minutes.

## Setup

This exercise is deliberately **workforce-unconstrained**: assume capacity can flex to any level via hiring/overtime, ignoring the physical `regular_capacity_units` / `overtime_capacity_units` ceiling stored in `supply_plan`. That ceiling is a real constraint — you'll bring it back in [Challenge 1](../challenges/challenge-01-constrained-sop-plan.md). For this exercise, use these flat parameters instead (distinct from `supply_plan`'s month-varying, capacity-capped numbers, on purpose):

```python
demand = [8200, 8600, 10400, 12800, 14200, 13000]   # Trail Footwear consensus_forecast_units, Jan-Jun
start_capacity = 8500          # December capacity, carried in from before the horizon
beginning_inventory = 1500
safety_stock_target = 900
regular_unit_cost = 22.00
overtime_unit_cost = 31.00
max_overtime_pct = 0.15        # of that month's regular capacity
hire_cost_per_unit = 45.00     # per unit of monthly capacity ADDED
layoff_cost_per_unit = 30.00   # per unit of monthly capacity REMOVED
holding_cost_per_unit_month = 3.00
```

Pull `demand` straight from your database to confirm it matches, rather than retyping it by hand:

```sql
SELECT consensus_forecast_units FROM demand_plan
WHERE product_family = 'Trail Footwear' ORDER BY month;
```

## Tasks

Write these as three functions (or three clearly-separated blocks) in `solutions.py`, each returning a `pandas.DataFrame` of the month-by-month simulation plus a total cost.

1. **Chase.** Capacity equals demand exactly every month, starting from `start_capacity`. Track hire/layoff units per month, production cost, and inventory (which should stay flat, since production always equals demand). *(Expected: total hire units across the horizon = 6,000; total layoff units = 1,500.)*

2. **Level.** Capacity is constant at the six-month average demand, held flat all six months after one initial hire from `start_capacity`. Track ending inventory month by month; flag any month where it drops below `safety_stock_target`, and any month it goes negative. *(Expected: the level capacity rate rounds to 11,200/month exactly — sanity check `sum(demand) / 6`.)*

3. **Mixed.** Design your own capacity ramp — pick 2–3 months where you add capacity (write down your reasoning for *why* those months), and use overtime (capped at `max_overtime_pct` of that month's regular capacity) to cover any remaining shortfall in other months. Track hire units, overtime units, and ending inventory the same way.

4. **Compare.** Build a small summary table with one row per strategy: `total_hire_cost`, `total_layoff_cost`, `total_production_cost` (regular + overtime), `total_holding_cost`, and `grand_total`. Print it sorted from cheapest to most expensive.

## Expected result (spot checks)

- **Chase total cost ≈ $1,820,400** (hire $270,000 + layoff $45,000 + production $1,478,400 + holding $27,000).
- **Level total cost ≈ $1,691,700** (one-time hire $121,500 + production $1,478,400 + holding $91,800) — Level's ending inventory should never go below the safety-stock target; its lowest point is June at 1,500 units.
- **Level should beat Chase** by roughly **$128,700 (≈7%)** — the same direction Lecture 2's smaller illustrative example showed, and for the same reason: hiring/layoff cost here is expensive relative to holding cost.
- A reasonable **Mixed** plan lands **between the two**, typically much closer to Level than to Chase — one workable ramp (capacity Jan 8,500 → Mar 10,200 → May 12,200, full overtime whenever there's a shortfall) totals **≈ $1,707,930**, about 1% above Level and 6% below Chase, with two months (April, May) dipping below the safety-stock target but never going fully negative.

If your Mixed total is meaningfully *higher* than Chase, that's a real, useful finding, not a bug — it means your ramp under-invests in capacity too early and leans on expensive overtime for too long. Try shifting a hire one month earlier and rerun.

## Done when…

- [ ] `solutions.py` runs top to bottom and prints all three DataFrames plus the comparison table.
- [ ] Your Chase and Level totals match the spot checks within a few dollars (small rounding differences are fine; large gaps mean a logic bug — check whether you're double-counting the one-time hire cost in Level, a common mistake).
- [ ] You have one sentence per strategy explaining, in your own words, *why* it costs what it costs.
- [ ] You can say which strategy you'd actually recommend to Crunch Gear's operations VP, and why — cheapest-on-paper is a valid answer, but so is "Mixed, because Level's June buffer is thinner than I'm comfortable with going into an unpredictable July," as long as you say why.

## Stretch

- Rerun Level with `safety_stock_target` raised to 1,500 (Trail Footwear's beginning inventory level) instead of 900. Does Level still never breach it? If it does, what's the cheapest fix — a higher constant capacity, or a small amount of overtime in the worst month?
- Plot all three strategies' `ending_inventory` over the six months on one line chart. Which strategy has the widest swing, and does that match your intuition from the cost breakdown?

## Submission

Commit `solutions.py` to your portfolio under `c40-week-10/exercise-02/`.
