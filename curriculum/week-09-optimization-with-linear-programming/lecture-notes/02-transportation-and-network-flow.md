# Lecture 2 — Transportation & Network Flow

> **Duration:** ~2 hours. **Outcome:** You can formulate the classic transportation problem for a real plant-to-DC network, solve it in PuLP reading data straight from SQL, and correctly interpret the solver's shadow prices as the marginal dollar value of one more unit of capacity or demand.

The single most common optimization problem in supply chain doesn't have a fancy name in daily use — everyone just calls it "the shipping plan." Formally, it's the **transportation problem**: multiple sources with limited supply, multiple destinations with required demand, a cost to move one unit across each source-destination pair, and the question "how much should flow across each lane to satisfy everyone at minimum total cost?" It was one of the first problems ever solved with linear programming (Hitchcock, 1941; Koopmans, 1947) and it's still the backbone of network-flow software used at every major logistics company today.

## 1. The problem, in Crunch Gear's words

Crunch Gear produces at three locations and ships to three regional distribution centers. This is the seed network from the [week README](../README.md) — make sure you've run that setup before continuing.

**Plants (supply):**

| Plant | Monthly capacity (units) |
|---|---:|
| El Paso Plant (`EP`) | 4,200 |
| Guadalajara CMT (`GDL`) | 5,800 |
| Ho Chi Minh CMT (`HCM`) | 5,000 |
| **Total supply** | **15,000** |

**Distribution centers (demand):**

| DC | Monthly demand (units) |
|---|---:|
| Austin East DC (`AUS`) | 6,000 |
| Memphis DC (`MEM`) | 5,000 |
| Reno DC (`RNO`) | 4,000 |
| **Total demand** | **15,000** |

**Shipping cost per unit, by lane ($):**

| Plant \ DC | AUS | MEM | RNO |
|---|---:|---:|---:|
| EP | 4 | 6 | 5 |
| GDL | 3 | 7 | 8 |
| HCM | 9 | 8 | 6 |

Total supply (15,000) exactly equals total demand (15,000) — this network is **balanced**. That's a deliberate teaching simplification; Section 6 covers what changes when it isn't.

## 2. Formulating the transportation problem

**Decision variables.** Let $x_{ij}$ = units shipped from plant $i$ to DC $j$, for every plant-DC pair. With 3 plants and 3 DCs that's **nine** decision variables — $x_{EP,AUS}, x_{EP,MEM}, \dots, x_{HCM,RNO}$. This is the pattern to internalize: a transportation problem has one decision variable **per lane**, not per node.

**Objective — minimize total shipping cost:**

$$\text{minimize } Z = \sum_{i} \sum_{j} c_{ij} \, x_{ij}$$

where $c_{ij}$ is the per-unit cost on lane $(i,j)$ from the table above. Written out, that's a sum of nine terms — tedious by hand, trivial for a solver.

**Supply constraints — can't ship more than a plant can make**, one per plant:

$$\sum_{j} x_{ij} \le \text{capacity}_i \quad \text{for every plant } i$$

**Demand constraints — must fully satisfy every DC**, one per DC:

$$\sum_{i} x_{ij} \ge \text{demand}_j \quad \text{for every DC } j$$

**Non-negativity:** $x_{ij} \ge 0$ for every lane.

Notice the direction of each inequality: supply is a **ceiling** (`≤`, you can't exceed capacity), demand is a **floor** (`≥`, you must at least meet it — a real DC can't be told "sorry, 200 units short this month"). When total supply exactly equals total demand, as here, both sets of constraints end up binding at the optimum automatically — but writing them as inequalities rather than equalities is the safer, more general habit, because it still gives a sensible (and *feasible*) answer if the numbers in a seed table ever drift out of balance.

## 3. Solving it in PuLP

The clean way to build a many-variable LP like this is with a **dictionary of variables** indexed by lane, generated in a loop — not nine hand-typed variable names.

```python
from pulp import LpProblem, LpVariable, LpMinimize, lpSum, LpStatus, value

plants = {"EP": 4200, "GDL": 5800, "HCM": 5000}
dcs    = {"AUS": 6000, "MEM": 5000, "RNO": 4000}

cost = {
    ("EP","AUS"): 4, ("EP","MEM"): 6, ("EP","RNO"): 5,
    ("GDL","AUS"): 3, ("GDL","MEM"): 7, ("GDL","RNO"): 8,
    ("HCM","AUS"): 9, ("HCM","MEM"): 8, ("HCM","RNO"): 6,
}

prob = LpProblem("crunch_gear_transportation", LpMinimize)

# One variable per (plant, dc) lane
x = {(p, d): LpVariable(f"ship_{p}_{d}", lowBound=0) for p in plants for d in dcs}

# Objective: total shipping cost across all lanes
prob += lpSum(cost[p, d] * x[p, d] for p in plants for d in dcs), "total_shipping_cost"

# Supply constraints — one per plant
for p in plants:
    prob += lpSum(x[p, d] for d in dcs) <= plants[p], f"supply_{p}"

# Demand constraints — one per DC
for d in dcs:
    prob += lpSum(x[p, d] for p in plants) >= dcs[d], f"demand_{d}"

prob.solve()
print("Status:", LpStatus[prob.status])
for (p, d), var in x.items():
    if var.value() > 0:
        print(f"{p} -> {d}: {var.value():.0f} units")
print("Total cost: $", value(prob.objective))
```

Solved output:

```
Status: Optimal
EP  -> AUS: 200 units
EP  -> MEM: 4000 units
GDL -> AUS: 5800 units
HCM -> MEM: 1000 units
HCM -> RNO: 4000 units
Total cost: $ 74200.0
```

**Read this as a shipping plan, not just numbers.** Guadalajara ships its entire 5,800-unit capacity to Austin East (its cheapest lane at \$3/unit — a big comparative advantage). El Paso splits: 4,000 units to Memphis and the remaining 200 topping off Austin East. Ho Chi Minh splits: 4,000 to Reno and the remaining 1,000 to Memphis, filling in what El Paso couldn't fully cover. Total network cost: **\$74,200/month**. Every lane not listed above (e.g., `HCM -> AUS`, `GDL -> MEM`) carries **zero** flow — the solver decided it's never worth using, even though it's a legal option.

## 4. Reading data from SQL instead of hard-coding it

Hard-coded dictionaries are fine for learning the shape of the model. In practice, this data lives in the tables you seeded in the [week README](../README.md), and you build the model by querying them:

```python
import sqlite3
import pandas as pd
from pulp import LpProblem, LpVariable, LpMinimize, lpSum, LpStatus, value

conn = sqlite3.connect("crunch_network.db")

plants_df = pd.read_sql("SELECT plant_id, monthly_capacity_units FROM plants", conn)
dcs_df    = pd.read_sql("SELECT dc_id, monthly_demand_units FROM distribution_centers", conn)
cost_df   = pd.read_sql("SELECT plant_id, dc_id, cost_per_unit FROM lane_costs", conn)

plants = dict(zip(plants_df.plant_id, plants_df.monthly_capacity_units))
dcs    = dict(zip(dcs_df.dc_id, dcs_df.monthly_demand_units))
cost   = {(r.plant_id, r.dc_id): r.cost_per_unit for r in cost_df.itertuples()}

# ... the rest of the model is IDENTICAL to Section 3 — that's the whole point.
# The formulation doesn't care where the numbers came from.
```

This is the pattern the mini-project builds out fully, including writing the solved flows back into a results table. Once a model is built from a query instead of a hard-coded dict, re-running it after next month's demand forecast changes is a five-minute job, not a rewrite.

## 5. Shadow prices — the solver's second answer

Every LP solve gives you two things, not one: the **primal solution** (the $x_{ij}$ values — the shipping plan) and the **dual solution** (a shadow price for every constraint — what each limit is *costing* or *saving* you). PuLP exposes shadow prices through each constraint's `.pi` attribute after solving:

```python
for name, constraint in prob.constraints.items():
    print(f"{name}: shadow price = {constraint.pi}, slack = {constraint.slack}")
```

A **shadow price** answers a precise question: *"if I relaxed this one constraint by exactly one unit, how much would the optimal objective value change?"* For a `≤` capacity constraint, the shadow price is the most you should be willing to pay for one more unit of that capacity. For a `≥` demand constraint, it's the marginal cost of that DC's last unit of required demand.

All three plants happen to be **fully used** at this optimum (4,200 + 5,800 + 5,000 = 15,000 = total supply, no leftover capacity anywhere), so don't assume "zero slack" and "zero shadow price" are the same thing — they aren't. A constraint can be binding (zero slack) and still carry a **zero** shadow price, which is exactly what happens to one of the three plants here. Working out the dual by hand (the classic transportation "potentials" method, **MODI**, if you want the mechanical procedure) gives:

| Constraint | Shadow price (magnitude) | Interpretation |
|---|---:|---|
| `supply_GDL` | \$3 | The single most valuable unit of capacity in the network — one more unit at Guadalajara would **save** \$3, because it's the cheapest source into Austin East and fully committed. |
| `supply_EP` | \$2 | One more unit of El Paso capacity would save \$2 — real value, but less than Guadalajara's. |
| `supply_HCM` | \$0 | Zero. Even though Ho Chi Minh is also fully used, more capacity there wouldn't help — it's the network's most expensive source, kept only because the cheaper plants are already maxed out, not because it's ever the *marginal* choice. |
| `demand_MEM` | \$8 | The most expensive unit of demand to satisfy — Memphis has no cheap dedicated source, so its next unit costs the most. |
| `demand_AUS`, `demand_RNO` | \$6 each | Cheaper on the margin than Memphis, but still real — read the same way: the cost of the *next* unit of demand at that DC, given everything else fixed. |

**Why this matters more than the shipping plan itself.** The shipping plan tells you what to do this month. The shadow prices tell you where to invest *next* month: Guadalajara's \$3 shadow price says "it's worth up to \$3/unit to expand this plant" — a concrete number you can compare against the actual cost of a capacity expansion. Ho Chi Minh's **zero** shadow price says the opposite: expanding your most expensive plant is a waste of capital as long as this cost structure holds, no matter how strategically important that site feels.

**A caution the exercise makes you rediscover:** a shadow price of zero on a *binding* constraint only promises that a small **increase** in that resource is worthless — it says nothing reliable about what a **decrease** would cost. Ho Chi Minh's capacity is fully used; cutting it, rather than growing it, can absolutely raise total cost, sometimes sharply, because losing capacity can force the network to route through a much more expensive lane or leave demand unmet entirely. Shadow prices describe the slope at the *current* corner of the feasible region — they say nothing about what happens once you've moved far enough to land on a different corner (or off the feasible region altogether). Exercise 2 and the mini-project's scenario analysis both test whether you actually internalized this distinction, not just the vocabulary.

**One more detail if you inspect `.pi` yourself:** different solvers report the *sign* of a dual value differently depending on internal convention — PuLP/CBC's raw `.pi` output for a `<=` or `>=` constraint may not match the "canonical" signs shown above (which are normalized so every supply shadow price is `≤ 0` and every demand shadow price is `≥ 0`, matching the economic story: more capacity never hurts, more demand never helps). Don't fight the sign — read the **magnitude and which constraints are nonzero**, then confirm the direction empirically by perturbing a number and re-solving, which is the reliable way to pin down what a shadow price means in practice.

## 6. What changes when the network isn't balanced

Real networks are rarely perfectly balanced. Two adjustments, both trivial in PuLP because you already wrote the constraints as inequalities, not equalities:

- **Total supply > total demand.** Some plant capacity goes unused — normal, and the `≤` supply constraints simply won't all bind. The solver will show slack on at least one `supply_*` constraint.
- **Total supply < total demand.** The problem as stated is **infeasible** — demand cannot be met at all. Real practice: add a **dummy plant** with capacity equal to the shortfall and a very high cost (representing an emergency buy, expedite, or lost sale) on every lane out of it, so the solver can still find an answer and tells you exactly how much demand would go unmet and where.

```python
# Dummy source pattern for an under-supplied network
plants["DUMMY"] = shortfall_units
for d in dcs:
    cost[("DUMMY", d)] = 100000   # a cost high enough it's only ever used as a last resort
```

The solver will only route flow through `DUMMY` if there is truly no other way to balance the network — and the resulting flow tells you exactly which DC(s) would come up short.

## 7. Network flow, more generally

The plant→DC transportation problem is the simplest member of a much larger family called **network flow problems**. The same modeling pattern — nodes, arcs, a per-unit cost, capacity limits on arcs or nodes, and conservation of flow (what comes in must go out, adjusted for any local supply/demand) — extends to:

- **Multi-echelon networks**, adding a plant→DC→customer stage (three tiers instead of two — this week's mini-project extends toward this).
- **Transshipment**, where a node is both a destination and a source (a DC that receives from plants and also cross-ships to another DC).
- **Minimum-cost flow with arc capacities**, where a single lane itself has a maximum throughput (a rail line, a port), independent of the plant's total capacity.

All of these stay **linear** as long as unit costs are constant and nothing forces an on/off decision. The moment you need a yes/no decision baked into the model — *should this DC even exist* — you've left pure LP behind. That's Lecture 3.

## 8. Check yourself

- How many decision variables does a transportation problem with 4 plants and 5 DCs have?
- Why is the supply constraint written with `≤` and the demand constraint with `≥`, rather than both as `=`?
- In the Crunch Gear solve, why does Guadalajara ship *only* to Austin East and nothing else, even though it has three legal lanes?
- What question does a shadow price answer, in one sentence?
- If `supply_EP`'s shadow price came back as `$2`, what would that tell you about expanding El Paso's capacity?
- What's the standard trick for handling a network where total demand exceeds total supply?

Next: Lecture 3 adds binary decisions to this same modeling pattern — not "how much to ship" but "should this facility exist at all" — and shows exactly where pure LP stops being enough.

## Further reading

- **PuLP — constraints and duals:** <https://coin-or.github.io/pulp/main/pulp.html>
- **NEOS Guide — Transportation Problem:** <https://neos-guide.org/case-studies/tp/>
- **Wikipedia — Transportation theory (linear programming origin, Hitchcock & Koopmans):** <https://en.wikipedia.org/wiki/Transportation_theory_(mathematics)>
