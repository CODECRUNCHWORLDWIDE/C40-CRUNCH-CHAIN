# Exercise 2 — Run the Forecast & Inventory Policy

**Time:** ~2 hours · **Tools:** Python (pandas, numpy, scipy)

Using Exercise 1's `demand_history` table, forecast every one of the 24 SKU-region pairs, backtest the forecast, and compute an EOQ / safety stock / reorder point for each. This exercise's output — a `demand_forecast` table and an `inventory_policy` table — is what Exercise 3's network LP consumes.

## Task 1 — Forecast every SKU-region pair (40 min)

Implement Lecture 2 §2's `forecast_demand()` function and run it for target month `"2026-07"`. Store the result (24 rows: `region_id`, `sku_id`, `forecast`, `sigma`, `month`) as a `demand_forecast` table.

```sql
SELECT COUNT(*) FROM demand_forecast;   -- 24  (4 regions × 6 SKUs)
```

## Task 2 — Backtest it (30 min)

Implement Lecture 2 §2's `backtest_mape()` function. Report the overall MAPE, and also break it out **by region** and **by SKU** (two `GROUP BY`-style aggregations in pandas):

```python
# by region
hist.groupby("region_id").apply(lambda g: ...)  # same leave-one-out logic, scoped to one region
```

Write your MAPE breakdown to `backtest_results.md` as two small tables. Answer, in 2–3 sentences: is forecast error meaningfully different across regions or SKUs, or is it roughly uniform? Given the demand generator applies the same 8% noise scale to every SKU-region, what would you *expect* the answer to be — and does your result match that expectation?

## Task 3 — Compute inventory policy (50 min)

Implement Lecture 2 §3's `compute_inventory_policy()` function. You'll need a `dc_assignment` dict mapping each region to a DC — for this exercise, use the **naive current-state assignment** from Lecture 1 §5 (`{"NE": "AUS", "SE": "AUS", "MW": "AUS", "WE": "REN"}` — note Memphis gets nothing yet; that changes in Exercise 3). Use `unit_cost=25.0` (the blended average) for every SKU, per Lecture 2's simplification.

Store the result as an `inventory_policy` table (24 rows: one EOQ / safety stock / reorder point per SKU-region).

```sql
SELECT COUNT(*) FROM inventory_policy;   -- 24
SELECT region_id, sku_id, eoq, safety_stock, reorder_point
FROM inventory_policy
WHERE region_id = 'MW' AND sku_id = 'SKU-100';
```

## Expected outcome

Task 3's SKU-100 / Midwest row should closely match Lecture 2 §3's hand-worked example — but note that exercise uses the **naive** DC assignment (Midwest → AUS, not MEM), so your lead time is `LEAD_TIME_DAYS["AUS"] = 6` days, not MEM's 4 days. Expect:

| Field | Approx. expected value |
|---|---:|
| `eoq` | ~1,321 units |
| `safety_stock` | slightly higher than Lecture 2's 77-unit MEM example (longer lead time → larger `σ_LT`) |
| `reorder_point` | correspondingly higher than Lecture 2's 290-unit example |

If your `eoq` is off, first check `annual_demand = forecast × 12` and `H = unit_cost × HOLDING_RATE` — a swapped order of operations here is the most common bug. If your `safety_stock` is off, check that `sigma_lt` scales by `√(lead_time_months)`, not `lead_time_months` directly (a very easy exponent-vs-no-exponent mistake).

## Deliverable

A directory `exercise-02/` containing `forecast.py`, `backtest_results.md`, and `inventory_policy.py`, with the `demand_forecast` and `inventory_policy` tables loaded into your database.

## Rules

- Backtest before trusting the forecast — this is not optional even though the underlying data is synthetic; the discipline is the point.
- Every number must trace back to a function in your `.py` files — no manually typed forecast or policy values.
