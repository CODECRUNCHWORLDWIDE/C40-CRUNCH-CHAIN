# Week 9 Exercises — Overview

Three exercises, worked in order. Each builds directly on the lecture that precedes it — do the lecture first, the exercise second.

| # | Exercise | Builds on | Time |
|--:|----------|-----------|-----:|
| 1 | [exercise-01-solve-an-lp-with-pulp.md](./exercise-01-solve-an-lp-with-pulp.md) | Lecture 1 — LP formulation + PuLP | 1h |
| 2 | [exercise-02-transportation-problem.md](./exercise-02-transportation-problem.md) | Lecture 2 — transportation model + shadow prices | 1.5h |
| 3 | [exercise-03-facility-location-model.md](./exercise-03-facility-location-model.md) | Lecture 3 — binary variables + MIP | 1.5h |

## How to work them

1. **Formulate on paper (or in a comment block) before you write code.** Every exercise below asks you to name decision variables, write the objective, and list constraints *before* touching PuLP. This is not busywork — formulation is the actual skill; the `.solve()` call is one line.
2. **Run the solver and check `LpStatus`.** Never trust a numeric answer without first confirming the solve was `"Optimal"`.
3. **Sanity-check the answer against the raw numbers.** Does the total cost look plausible given the cost table? Do all the constraints you wrote down actually hold for the returned values? A solver can't catch a formulation bug — only you can.
4. **Save your code.** Each exercise below tells you the expected filename and where it lives in your portfolio.

## Environment check

Before Exercise 1, confirm your solver stack works:

```python
import pulp
print(pulp.listSolvers(onlyAvailable=True))   # must include 'PULP_CBC_CMD'

from scipy.optimize import linprog
print(linprog([1], A_ub=[[1]], b_ub=[10]).success)   # should print True
```

If either import fails, revisit [`resources.md`](../resources.md) before continuing.
