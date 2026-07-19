# Challenge 2 — Capacitated Production Plan

**Time:** ~90 minutes. **Difficulty:** Medium-hard. **Validate with the cumulative-balance check below — no answer key provided.**

## The scenario

Every model so far this week has been **single-period** — one month, solve once, done. Real production planning has to decide **when**, across many periods, to build inventory ahead of a demand spike that a single period's capacity can't cover alone. This challenge is the multi-period cousin of Lecture 1's production-mix LP: instead of splitting *one* period's capacity across *two products*, you're splitting *one product's* production across *four quarters*, deciding how much to build early and carry in inventory versus how much to build just in time.

Crunch Gear's El Paso plant needs a **4-quarter production plan** for a single high-volume item, the **Trailhead Jacket**, given a capacity limit that varies by quarter (planned maintenance downtime in Q3–Q4) and a demand forecast that peaks in Q3.

**Demand and capacity by quarter (units):**

| Quarter | Demand | Capacity |
|---|---:|---:|
| Q1 | 3,000 | 5,000 |
| Q2 | 5,000 | 5,000 |
| Q3 | 6,000 | 4,000 |
| Q4 | 4,000 | 4,000 |
| **Total** | **18,000** | **18,000** |

**Costs:**

- Production cost: **$40/unit**, the same in every quarter (assume no cost inflation across the year).
- Holding cost: **$3/unit**, charged on whatever inventory is left **at the end of** each quarter (i.e., inventory built in Q1 that's still on the shelf at the end of Q1, Q2, and Q3 before finally being sold in Q4 costs $3 three separate times — three quarter-ends of holding).
- **No backorders allowed** — every quarter's demand must be met from that quarter's production plus carried-in inventory; you may never ship negative inventory.
- Starting inventory (beginning of Q1) is **zero**. Ending inventory is not required to be zero, but holding cost will make the solver want it that way if it's cheap to arrange.

## Your task

1. **Formulate the multi-period LP.** You need two families of decision variables:
   - $p_t \ge 0$ — units **produced** in quarter $t$, for $t \in \{1,2,3,4\}$.
   - $I_t \ge 0$ — units of **inventory on hand at the end of** quarter $t$.

   Write the **inventory balance constraint** — the equation that ties one quarter to the next — for every quarter:

   $$I_t = I_{t-1} + p_t - \text{demand}_t$$

   (with $I_0 = 0$, the given starting inventory). Then the **capacity constraint**, one per quarter:

   $$p_t \le \text{capacity}_t$$

   And the objective — minimize total production cost plus total holding cost:

   $$\text{minimize } Z = \sum_t 40\, p_t + \sum_t 3\, I_t$$

2. **Build it in PuLP.** The inventory balance is an **equality** constraint (`==`), not an inequality — get this wrong and the model will let inventory appear from nowhere.

3. **Solve and report**, quarter by quarter: production, ending inventory, and running total cost.

4. **Explain the shape of the answer** in one paragraph: which quarter(s) build up inventory ahead of the Q3 shortfall, and does the solver produce the absolute earliest it possibly could, or does it wait as long as feasible? Why does the $3 holding cost push it toward one behavior over the other?

## How to validate your answer (cumulative-balance check)

Before you even solve the LP, you can derive two facts from the raw numbers alone — use them to sanity-check your solver output, not to skip building the model:

1. **Total production across all 4 quarters must equal exactly 18,000** — total demand equals total capacity for the year, so there is no slack anywhere in aggregate; nothing can be short, and (with positive holding cost) nothing beneficial is produced in excess. If your solved `p_1 + p_2 + p_3 + p_4` isn't exactly 18,000, you have a bug.
2. **Compute cumulative capacity and cumulative demand through each quarter** and confirm cumulative capacity is always ≥ cumulative demand (the feasibility condition for a no-backorder plan):

   | Through | Cumulative demand | Cumulative capacity |
   |---|---:|---:|
   | Q1 | 3,000 | 5,000 |
   | Q2 | 8,000 | 10,000 |
   | Q3 | 14,000 | 14,000 |
   | Q4 | 18,000 | 18,000 |

   Notice cumulative capacity exactly **equals** cumulative demand at Q3 and at Q4 — zero slack at those two checkpoints. That means your solved model **must** show ending inventory of exactly **zero** after Q3 and after Q4 — there's no room for anything else. If your solver reports positive inventory remaining after Q3 or Q4, something in your balance constraints is wrong.

## Constraints

- Production cost is constant per unit and total production is fixed by the arithmetic above — the *only* real lever the optimizer has is **timing** (when to hold inventory), since total production cost is invariant regardless of the schedule chosen. Say so explicitly in your written explanation; realizing this before you solve is exactly the kind of formulation insight this week is training.
- No backorders — don't add a variable that lets a quarter ship more than it has produced-plus-carried.

## Hints

<details>
<summary>On writing the balance constraint for Q1</summary>

$I_0$ (starting inventory) is a **constant**, not a variable — it's given as zero. Write Q1's balance as `I[1] == 0 + p[1] - demand[1]`, not as a reference to some `I[0]` variable you never created. For Q2 onward, `I[t] == I[t-1] + p[t] - demand[t]` correctly chains to the *previous quarter's variable*.

</details>

<details>
<summary>On why this challenge is an LP, not a MIP</summary>

Nothing here forces an on/off decision — $p_t$ and $I_t$ are both naturally continuous quantities. This stays a pure LP. The **stretch** below is what would push it into MIP territory — notice the difference before you attempt it.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Formulation | Missing or wrong inventory balance (treats each quarter independently) | Correct chained equality constraint linking every quarter |
| Validation | Doesn't check total production or the Q3/Q4 zero-inventory condition | Both cumulative-balance checks run and reported before trusting the answer |
| Explanation | Restates the numbers without interpretation | Correctly explains that holding cost drives production as *late* as feasibility allows, and identifies exactly which quarter(s) must build ahead |
| Code | Balance constraint written with `<=` instead of `==` | Correct equality constraint, `I_0` handled as a constant not a phantom variable |

## Stretch — push it into MIP territory

Add a **setup cost**: producing anything at all in a given quarter costs an extra **$2,000 fixed charge**, regardless of volume (a real changeover/setup cost). This means $p_t$ can only be positive if a new binary variable $y_t$ (produce this quarter, yes/no) is 1 — exactly the Lecture 3 linking trick, applied to time periods instead of facilities:

$$p_t \le \text{capacity}_t \cdot y_t$$
$$\text{minimize } Z = \sum_t \left(40\, p_t + 2000\, y_t\right) + \sum_t 3\, I_t$$

Re-solve as a MIP. Does the solver now choose to skip producing in any quarter entirely (paying more holding cost in exchange for avoiding a $2,000 setup), or does the setup cost turn out too small to change the schedule? Report the new total cost and compare it to the pure-LP answer from Task 3.

## Submission

Commit `solution.py` and a short `notes.md` (your validation checks, written explanation, and — if attempted — the stretch MIP results) to your portfolio under `c40-week-09/challenge-02/`.
