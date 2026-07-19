# Exercise 3 — Build a Facility-Location Model

**Goal:** Implement the Lecture 3 capacitated facility-location MIP yourself, confirm the solver closes Austin East in favor of the Phoenix candidate, then use the LP relaxation to see exactly what integrality bought you.

**Estimated time:** 1.5 hours.

## The scenario (same as Lecture 3)

Four candidate DCs, three demand regions. Reference tables are in [Lecture 3](../lecture-notes/03-facility-location-and-mip.md#2-the-capacitated-facility-location-problem) if you need them again — re-type them into your script rather than copy-pasting the whole file; typing the numbers in yourself is part of building the habit of reading a cost table correctly.

Create a file `solution.py`.

## Tasks

1. **Formulate first**, as a comment block: name the binary variables, the continuous variables, the objective, and both families of constraints (demand, and capacity-gated-by-open).

2. **Build the full MIP in PuLP.** Binary `y_k` per candidate (`cat=LpBinary`), continuous `x_{k,r}` per (candidate, region) lane, objective = fixed cost of open sites + variable shipping cost, demand constraints (`≥`), and capacity constraints written as `sum(x) <= capacity_k * y_k`.

3. **Solve and report:**
   - Which candidates are open / closed.
   - The full shipping plan (every nonzero lane).
   - Total monthly cost.

4. **Solve the LP relaxation.** Rebuild the exact same model but with `y_k` as continuous (`lowBound=0, upBound=1`, no `cat=LpBinary`). Solve it and print the `y_k` values.

5. **Compare the two solves** in a short written note:
   - Does the relaxed model produce any fractional `y_k`? Which one(s)?
   - Is the relaxed objective value higher, lower, or equal to the MIP's objective? (Think about *why* before you check — a relaxation can never make a minimization problem's optimal value *worse* than the integer-constrained version. Confirm that holds here and explain why in one sentence.)
   - What would it mean operationally if you'd naively "rounded" the fractional `y_k` from the relaxation instead of solving the real MIP?

## Expected result

- MIP solve: **Austin East closed**; Memphis, Reno, and Phoenix all **open**. Total cost **$240,000/month**.
- Shipping plan: Reno → West (5,000), Memphis → Central (6,000), Phoenix → East (4,000).
- The LP relaxation's objective should be **less than or equal to** $240,000 (relaxing integrality can only help or tie, never hurt, a minimization problem) — and at least one `y_k` should come back fractional, demonstrating exactly why you can't skip the `cat=LpBinary`.

## Done when…

- [ ] The MIP solve matches the expected result (AE closed, MEM/RNO/PHX open, $240,000).
- [ ] The LP relaxation is built as a genuinely separate model (not just re-reading the MIP's rounded output).
- [ ] Your written comparison correctly explains the relaxation-bound relationship and states it in your own words, not copied from the lecture.

## Stretch

- Add a **single-sourcing constraint**: each demand region must be served by exactly one open candidate (no splitting a region's demand across multiple DCs — some real distribution contracts require this). This turns `x_{kr}` into something that needs its own binary "region `r` is served by candidate `k`" indicator, linked to the flow. Re-solve and see whether total cost goes up — and if so, by how much single-sourcing is "costing" the network. (Hint: you'll need a binary `z_{kr}` with `x_{kr} <= demand_r * z_{kr}` and `sum_k z_{kr} == 1` for each region.)
- Force Austin East to stay open (`y["AE"].setInitialValue(1)` won't do it — you need an actual constraint `y["AE"] == 1`) and re-solve. Report the cost penalty of keeping the incumbent versus the true optimum — a number worth having in your back pocket the next time "we've always run it this way" comes up in a real meeting.

## Submission

Commit `solution.py` to your portfolio under `c40-week-09/exercise-03/`.
