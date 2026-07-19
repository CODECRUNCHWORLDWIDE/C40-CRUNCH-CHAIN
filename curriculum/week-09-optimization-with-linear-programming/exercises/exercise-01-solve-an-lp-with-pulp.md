# Exercise 1 — Solve an LP with PuLP

**Goal:** Take a fresh production-mix scenario (different numbers from Lecture 1's jackets/backpacks example), formulate it from scratch, and solve it in PuLP. By the end, writing `LpProblem` → variables → objective → constraints → `.solve()` feels automatic.

**Estimated time:** 1 hour.

## The scenario

Crunch Gear's El Paso plant is also considering a run of two camping products: the **Summit Tent** and the **Basecamp Sleeping Bag**. Each consumes shared ripstop fabric and sewing-machine time, and each earns a different profit:

| Product | Ripstop fabric (yd) | Machine hours | Profit/unit |
|---|---:|---:|---:|
| Summit Tent (`tents`) | 6 | 5 | $70 |
| Basecamp Sleeping Bag (`bags`) | 3 | 2 | $30 |
| **Available this week** | **2,400 yd** | **1,800 hr** | |

## Setup

Confirm your solver stack:

```python
import pulp
print(pulp.listSolvers(onlyAvailable=True))
```

Create a file `solution.py`.

## Tasks

1. **Formulate first.** In a comment block at the top of `solution.py`, write:
   - The two decision variables and their units.
   - The objective function (maximize what, exactly?).
   - Every constraint, in words and then in symbols.

2. **Build the model in PuLP.** Follow the pattern from Lecture 1 Section 5:
   - Create an `LpProblem` with the right direction (`LpMaximize`).
   - Create the two decision variables with `lowBound=0`.
   - Add the objective as the *first* `+=`.
   - Add the fabric and machine-hour constraints.

3. **Solve and print the result.** Print `LpStatus`, both variable values, and the objective value.

4. **Verify by hand.** Plug your solved `tents` and `bags` values back into both constraint expressions. Confirm neither exceeds its limit, and note which constraint(s) are binding (zero slack) at the optimum.

5. **Sensitivity check.** Change the machine-hours limit from 1,800 to 2,200 and re-solve. Does the optimal product mix change? Does profit go up, and does that match your intuition about which resource was truly limiting?

## Expected result

- Optimal solution: **200 tents, 400 sleeping bags**, maximum weekly profit **$26,000**.
- Both the fabric constraint (`6·tents + 3·bags ≤ 2400`) and the machine-hours constraint (`5·tents + 2·bags ≤ 1800`) are binding — both used up exactly, zero slack.
- With machine hours raised to 2,200: the solver should shift the mix — verify what direction it moves and whether the *fabric* constraint alone now determines the answer.

## Done when…

- [ ] `solution.py` has the written formulation as a comment block, before any code.
- [ ] The PuLP model runs and reports `Status: Optimal`.
- [ ] Printed output matches the expected result above (200 tents, 400 bags, $26,000).
- [ ] You've hand-verified both constraints hold and identified which are binding.
- [ ] You ran the 2,200-hour sensitivity check and can explain the shift in one sentence.

## Stretch

- Solve the same problem in `scipy.optimize.linprog` (remember: negate the objective, and everything must be in `≤` form). Confirm you get the same `(200, 400)` answer.
- Add a third constraint: at most 350 sleeping bags can be produced this week (a packaging-supply limit). Re-solve and note whether the optimal solution actually changes — sometimes a new constraint doesn't bind at all.

## Submission

Commit `solution.py` to your portfolio under `c40-week-09/exercise-01/`.
