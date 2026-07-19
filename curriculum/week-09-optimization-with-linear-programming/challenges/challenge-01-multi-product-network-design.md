# Challenge 1 — Multi-Product Network Design

**Time:** ~90 minutes. **Difficulty:** Medium-hard. **No single "correct" number provided — validate with the bound described below.**

## The scenario

Lecture 2's transportation problem shipped one undifferentiated "unit" from plant to DC. Real networks ship a **product mix**, and the twist that makes this genuinely harder than solving two separate transportation problems is that **the products compete for the same plant capacity.** Crunch Gear's three plants now produce two products — **Rain Shells** and **Duffel Bags** — and each plant's capacity is expressed in shared **production-hours per month**, not "units," because the two products don't take the same amount of time to make.

**Plants — capacity in production-hours/month:**

| Plant | Capacity (hours/month) |
|---|---:|
| El Paso (`EP`) | 20,000 |
| Guadalajara (`GDL`) | 24,000 |
| Ho Chi Minh (`HCM`) | 18,000 |

**Hours consumed per unit (same at every plant):**

| Product | Hours/unit |
|---|---:|
| Rain Shell | 2.0 |
| Duffel Bag | 1.5 |

**Demand by DC and product (units/month):**

| DC | Rain Shells | Duffel Bags |
|---|---:|---:|
| Austin East (`AUS`) | 3,000 | 4,000 |
| Memphis (`MEM`) | 2,500 | 3,000 |
| Reno (`RNO`) | 2,000 | 2,500 |
| **Total** | **7,500** | **9,500** |

**Shipping cost per unit ($) — Rain Shells** (same lane costs as the Week 9 seed network):

| Plant \ DC | AUS | MEM | RNO |
|---|---:|---:|---:|
| EP | 4 | 6 | 5 |
| GDL | 3 | 7 | 8 |
| HCM | 9 | 8 | 6 |

**Shipping cost per unit ($) — Duffel Bags** (bags pack denser, so shipping runs about 70% of the Rain Shell rate on every lane — compute this yourself, don't hand-type a new table):

```python
rain_shell_cost = {
    ("EP","AUS"): 4, ("EP","MEM"): 6, ("EP","RNO"): 5,
    ("GDL","AUS"): 3, ("GDL","MEM"): 7, ("GDL","RNO"): 8,
    ("HCM","AUS"): 9, ("HCM","MEM"): 8, ("HCM","RNO"): 6,
}
duffel_bag_cost = {lane: round(c * 0.7, 2) for lane, c in rain_shell_cost.items()}
```

## Your task

1. **Formulate the multi-product transportation model.** Decision variables now need a **third index**: $x_{p,i,j}$ = units of product $p$ shipped from plant $i$ to DC $j$. Write out:
   - The objective (sum of shipping cost across every product, plant, and DC).
   - The **demand constraints** — now one per (product, DC) pair, not just per DC.
   - The **capacity constraint** — this is the genuinely new part. One constraint per plant, summing **hours consumed by both products together**:

     $$\sum_{j} \left( 2.0 \cdot x_{\text{RainShell},i,j} + 1.5 \cdot x_{\text{DuffelBag},i,j} \right) \le \text{capacity}_i \quad \text{for every plant } i$$

2. **Build and solve it in PuLP.** Reuse the loop-over-a-dict pattern from Lecture 2 Section 3, but index everything by `(product, plant, dc)` triples.

3. **Report the shipping plan by product** — two tables, Rain Shells and Duffel Bags separately — plus the total combined cost.

4. **Check plant utilization.** For each plant, compute total hours actually used (sum of both products' hour consumption) versus capacity. Which plant(s), if any, are fully utilized (zero slack)?

## How to validate your answer (no answer key provided)

There's no single "correct" total cost handed to you here — instead, run this **decomposition bound** to sanity-check your joint solution:

1. Solve the Rain Shell transportation problem **alone**, giving it a capacity split of each plant's hours converted to Rain-Shell-only units (`capacity_i / 2.0`).
2. Solve the Duffel Bag transportation problem **alone**, the same way (`capacity_i / 1.5`).
3. Add the two total costs together.

**Your joint multi-product solution's total cost must be less than or equal to this decomposed sum.** Why: the joint model has strictly more flexibility — it can trade hours between products wherever that's cheaper overall, while the decomposed version locks in an arbitrary capacity split before either problem even sees the cost data. If your joint solution comes back *more expensive* than the decomposed sum, you have a bug — go find it before trusting any other number in your solution.

## Constraints

- Every plant-DC-product combination is a legal lane (no forced restrictions) — the solver decides what to use.
- Report your plant-utilization table even for plants the solver leaves with slack; "this plant has spare capacity" is itself a useful finding.
- State explicitly which capacity split you used for the decomposition bound in Step 1–2 above, since that choice is somewhat arbitrary (an even split by current demand share is a reasonable default).

## Hints

<details>
<summary>On indexing three-dimensional variables in PuLP</summary>

Build the variable dictionary with a triple-nested comprehension, exactly the same pattern as the two-dimensional case:

```python
products = ["RainShell", "DuffelBag"]
x = {
    (p, i, j): LpVariable(f"ship_{p}_{i}_{j}", lowBound=0)
    for p in products for i in plants for j in dcs
}
```

Everything downstream — the objective sum, the constraint sums — is just one more `for p in products` layer wrapped around the two-dimensional version from Lecture 2.

</details>

<details>
<summary>On the capacity constraint</summary>

Don't write two separate capacity constraints per plant (one per product) — that would let each product use the *full* capacity independently, which double-counts the resource. There must be exactly **one** capacity constraint per plant, summing the hours consumed by *both* products together, as shown in the formulation above.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Formulation | Two separate capacity constraints per plant | One shared capacity constraint correctly summing both products' hour consumption |
| Correctness | Joint solution costs more than the decomposition bound | Joint solution ≤ decomposition bound, and you can explain why |
| Plant utilization | Not reported, or reported without units | Hours used vs. capacity, per plant, clearly labeled |
| Code structure | Hard-coded per-product logic duplicated | Single loop parameterized over `products`, no copy-pasted blocks |

## Submission

Commit `solution.py` and a short `notes.md` (your decomposition-bound check and plant-utilization table) to your portfolio under `c40-week-09/challenge-01/`.
