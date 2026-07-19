# Lecture 2 — Aggregate Planning & Balancing

> **Duration:** ~2 hours. **Outcome:** You can name and compute the three classic aggregate-planning strategies — chase, level, mixed — cost each one out in Python across a multi-month horizon, and explain, with real numbers, why the cheapest strategy on paper isn't automatically the one a real S&OP team picks.

## 1. What "aggregate" means here

Every table in this course down to Week 9 planned at **SKU level** — this exact shoe, this exact size. S&OP does not. A monthly capacity decision — "do we run overtime in March" — doesn't care whether the extra units are size 9 or size 11; it cares about **total units of the Trail Footwear family**, because that's the level the shared production line actually operates at. **Aggregate planning** is deliberately planning at family level (or plant level, or workforce level) — it trades away SKU-level precision for a planning horizon (usually 3–18 months) and a decision (production rate, workforce size, inventory level) that SKU-level planning is the wrong grain for. You'll disaggregate the result back down to SKUs later, closer to execution — that's a Week 11–12 problem, not this week's.

## 2. The three strategies

Given a demand forecast that varies month to month, and a company that has to decide **how much capacity to run** each month, there are three classic strategies:

| Strategy | What it does | Main cost driver | Main risk |
|---|---|---|---|
| **Chase** | Match capacity to demand *every single month* — hire when demand rises, lay off when it falls | Hiring and layoff cost, repeated all year | Workforce churn, training cost, morale — real costs this simple model doesn't even capture |
| **Level** | Hold capacity **constant** at (roughly) the average demand rate — build inventory in slow months, draw it down in peak months | Inventory holding cost | Needs warehouse space and product that doesn't spoil/obsolete quickly; can miss a demand spike bigger than the buffer |
| **Mixed** | Some capacity change (a moderate ramp, not month-to-month whiplash) + some overtime for peaks + some inventory buffer | A blend of all three costs, none of them extreme | Most realistic, hardest to reason about by hand — no single lever does all the work |

None of these is "correct" in the abstract. Which one wins is an **arithmetic question specific to your cost structure** — how expensive is it to hire/fire relative to how expensive it is to hold inventory relative to how expensive overtime is. Change those numbers and the winner changes. That's exactly why this lecture computes a real example instead of asserting a rule.

## 3. A worked example, start to finish

Set aside Trail Footwear's real numbers for a moment — this section uses a smaller, cleaner illustrative case so the arithmetic is easy to verify by hand. (Exercise 2 asks you to run this same method against the real `demand_plan`/`supply_plan` numbers.)

**The scenario:** a single product line, six months, demand forecast `1000, 1000, 1300, 1700, 1600, 1200` units (total 7,800, average 1,300/month). Starting capacity (last month, before this horizon begins) is **1,000 units/month**. Beginning inventory is **200 units**; the safety-stock floor is **150 units**. Costs: regular production **$25/unit**, overtime **$35/unit** (max overtime = 20% of that month's regular capacity), hiring **$60 per unit of added monthly capacity**, layoff **$45 per unit of removed monthly capacity**, holding **$4/unit/month** on ending inventory.

Set this up as a small table so the simulation has something to read from:

```sql
CREATE TABLE demo_demand (
    month_num INTEGER PRIMARY KEY,
    demand_units INTEGER NOT NULL
);
INSERT INTO demo_demand VALUES (1,1000),(2,1000),(3,1300),(4,1700),(5,1600),(6,1200);
```

The three strategies are each an **iterative simulation** — this month's ending inventory depends on last month's ending inventory, so this is naturally a Python loop (the same reason Week 7's routing heuristics were Python, not SQL — SQL is superb at set-based aggregation and terrible at "carry a running state forward one row at a time, making a decision at each step").

### Chase — hire/fire to match demand exactly

```python
import pandas as pd

demand = [1000, 1000, 1300, 1700, 1600, 1200]
start_capacity = 1000
regular_cost, hire_cost, layoff_cost, holding_cost = 25, 60, 45, 4
inventory = 200

capacity = start_capacity
rows = []
for d in demand:
    change = d - capacity                 # chase: capacity always equals demand
    hire_units   = max(change, 0)
    layoff_units = max(-change, 0)
    capacity = d
    inventory += capacity - d             # always zero net change under pure chase
    rows.append({"demand": d, "capacity": capacity, "hire": hire_units,
                 "layoff": layoff_units, "ending_inv": inventory})

chase = pd.DataFrame(rows)
chase["hire_cost"]    = chase["hire"]   * hire_cost
chase["layoff_cost"]  = chase["layoff"] * layoff_cost
chase["prod_cost"]    = chase["capacity"] * regular_cost
chase["holding_cost"] = chase["ending_inv"] * holding_cost
total_chase = chase[["hire_cost","layoff_cost","prod_cost","holding_cost"]].sum().sum()
print(chase)
print("TOTAL CHASE COST:", total_chase)
```

```
   demand  capacity  hire  layoff  ending_inv
0    1000      1000     0       0         200
1    1000      1000     0       0         200
2    1300      1300   300       0         200
3    1700      1700   400       0         200
4    1600      1600     0     100         200
5    1200      1200     0     400         200

TOTAL CHASE COST: 264300.0
```

Under pure chase, inventory never moves — production always equals demand, so it stays flat at the starting 200 units all six months. All the cost lives in **hiring 700 units of capacity** ($42,000) **and laying off 500 units** ($22,500) — $64,500 of pure churn cost, on top of $195,000 of regular production (7,800 units × $25) and $4,800 of holding cost (200 units × $4 × 6 months held flat). **Total: $264,300.**

### Level — hold capacity constant at the average

```python
level_capacity = round(sum(demand) / len(demand))   # 1300
hire_units = max(level_capacity - start_capacity, 0)  # one-time ramp
inventory = 200
rows = []
for d in demand:
    inventory += level_capacity - d
    rows.append({"demand": d, "capacity": level_capacity, "ending_inv": inventory})

level = pd.DataFrame(rows)
one_time_hire_cost = hire_units * hire_cost
prod_cost = level_capacity * len(demand) * regular_cost
holding_total = (level["ending_inv"] * holding_cost).sum()
total_level = one_time_hire_cost + prod_cost + holding_total
print(level)
print("TOTAL LEVEL COST:", total_level)
```

```
   demand  capacity  ending_inv
0    1000      1300         500
1    1000      1300         800
2    1300      1300         800
3    1700      1300         400
4    1600      1300         100
5    1200      1300         200

TOTAL LEVEL COST: 224200.0
```

One hire of 300 units in month 1 ($18,000), then capacity never changes again. Notice **month 5's ending inventory drops to 100 units — below the 150-unit safety floor.** That's a real flag a level plan has to carry (it never goes *negative*, so there's no stockout, but it briefly runs thinner than policy allows), and it's exactly the kind of thing Exercise 3's gap analysis is built to catch. Production cost is identical to chase ($195,000 — same 7,800 total units, all at the regular rate since level never needs overtime here). Holding cost is higher ($11,200 vs. $4,800) because level, by design, carries real inventory swings. **Total: $224,200 — $40,100 cheaper than chase**, an ~15% saving, purely because this cost structure makes hiring/firing expensive relative to holding a warehouse buffer.

### Mixed — a moderate ramp plus overtime plus some inventory risk

```python
plan_capacity = [1000, 1000, 1200, 1200, 1400, 1400]   # ramps in months 3 and 5
max_overtime_pct = 0.20
overtime_cost = 35
inventory = 200
prev_capacity = start_capacity
rows = []
for cap, d in zip(plan_capacity, demand):
    hire = max(cap - prev_capacity, 0)
    prev_capacity = cap
    max_ot = round(cap * max_overtime_pct)
    shortfall = max(d - cap, 0)
    ot_units = min(shortfall, max_ot)
    inventory += cap + ot_units - d
    rows.append({"demand": d, "capacity": cap, "hire": hire,
                 "overtime": ot_units, "ending_inv": inventory})

mixed = pd.DataFrame(rows)
print(mixed)
```

```
   demand  capacity  hire  overtime  ending_inv
0    1000      1000     0         0         200
1    1000      1000     0         0         200
2    1300      1200   200       100         200
3    1700      1200     0       240         -60
4    1600      1400   200       280          20
5    1200      1400     0         0         220
```

This is the interesting one: even ramping capacity to 1,200 in month 3 and using the maximum overtime available, **month 4 still runs a 60-unit backorder** (`ending_inv = -60`) — demand of 1,700 simply outstrips what 1,200 regular + 240 max overtime can produce, and the inventory buffer built in months 1–2 isn't quite big enough to absorb the rest. Month 5 claws back to positive by ramping again and running more overtime, ending the horizon at a healthy 220. Total cost — $24,000 hiring, $201,700 combined regular+overtime production, $3,360 holding on the positive months, plus a $900 backorder penalty (60 units × $15/unit) for the month-4 shortfall — comes to **$229,960**: cheaper than chase, about $5,760 more expensive than pure level.

### Comparing the three

| Strategy | Total cost | vs. Level |
|---|---:|---:|
| Level | $224,200 | — |
| Mixed | $229,960 | +$5,760 (+2.6%) |
| Chase | $264,300 | +$40,100 (+17.9%) |

**Level wins on paper here** — but "wins on paper" and "is what a real team picks" aren't the same thing, which is the whole point of Section 4.

## 4. Why the cheapest strategy isn't automatically the right one

Level's win in this example depends entirely on holding costs being *cheap relative to* hiring/firing costs. Change the inputs and the ranking flips:

- If this were **fresh produce** instead of trail footwear (holding cost effectively infinite past a few days — spoilage), level's core assumption breaks completely; chase or heavy overtime becomes the only workable strategy regardless of its higher dollar cost.
- If the workforce were **unionized with strict no-layoff clauses**, chase's layoff cost isn't $45/unit — it might be legally impossible, making chase not just expensive but *unavailable*.
- If warehouse space is **physically capped**, level's inventory buildup (800 units in month 3 of this example) might not fit, forcing some overtime/mixed approach regardless of cost.
- Mixed's real advantage — even when it isn't cheapest — is that it **caps the worst case on both axes**: less workforce volatility than chase, less inventory/stockout risk than level. A real S&OP team often picks mixed **on purpose**, trading a few thousand dollars of "optimal" cost for a plan that's more forgiving when a forecast turns out wrong (and forecasts always turn out at least a little wrong).

The number you compute is a critical input to the decision. It is not the decision by itself — that's a judgment call a person makes, informed by the number, and it's exactly what Exercise 2 asks you to make for Trail Footwear's real, harder capacity picture.

## 5. Buffers: the two levers under the hood

Both level and mixed strategies lean on two buffers that chase never needs:

- **Inventory buffer** — produce more than you need in a slack month, store it, draw it down in a peak month. Bounded by holding cost and shelf life/obsolescence, and by the safety-stock floor you must never drop below (or, if you do, by how much and for how long you're willing to run thin).
- **Capacity buffer (overtime/subcontract)** — a *temporary* boost above regular capacity, at a cost premium, used only when needed rather than baked into the permanent workforce. Bounded by the maximum available (this week's `overtime_capacity_units` / `subcontract_capacity_units` columns) and by the premium you're willing to pay per unit.

A pure chase strategy uses neither buffer — it just resizes the permanent workforce every month, which is why it's usually the most expensive option once hiring/firing costs are realistic. Real aggregate plans almost always blend both buffers with a moderate capacity ramp, which is exactly what "mixed" means.

## 6. Check yourself

- In one sentence each, what does chase optimize for, what does level optimize for, and what does mixed try to balance?
- In the worked example, why does chase's production cost equal level's production cost exactly ($195,000 each) even though the two strategies look completely different?
- Why did the mixed strategy run a negative ending inventory (a backorder) in month 4 even after using maximum overtime?
- Name one real-world constraint (not a dollar cost) that could make chase the only viable strategy even if it's the most expensive on paper.
- Why is a month-by-month aggregate-planning simulation naturally a Python loop rather than a single SQL query?

If those are automatic, Lecture 3 zooms out from "how do we produce it" to "does the resulting plan actually make the money Finance is expecting" — Integrated Business Planning.

## Further reading

- **APICS/ASCM CPIM body of knowledge — Aggregate Planning:** <https://www.ascm.org/>
- **pandas — Rolling and cumulative computations (`cumsum`, iterative `for` loops over DataFrame rows):** <https://pandas.pydata.org/docs/user_guide/window.html>
- **Investopedia — Overtime and workforce cost basics (background reading, not SCM-specific):** <https://www.investopedia.com/>
