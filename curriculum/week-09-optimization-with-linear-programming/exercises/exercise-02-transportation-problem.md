# Exercise 2 — Solve the Transportation Problem

**Goal:** Build the Lecture 2 plant→DC transportation model yourself, reading data from the Week 9 SQL seed instead of a hard-coded dict, and confirm both the shipping plan and the shadow prices match what the lecture derived.

**Estimated time:** 1.5 hours.

## Setup

You already ran the seed from the [week README](../README.md). Confirm it:

```sql
SELECT SUM(monthly_capacity_units) FROM plants;             -- 15000
SELECT SUM(monthly_demand_units) FROM distribution_centers; -- 15000
SELECT COUNT(*) FROM lane_costs;                             -- 9
```

Create a file `solution.py`.

## Tasks

1. **Load the network from SQL**, not a hard-coded dict. Connect with `sqlite3` (or `psycopg2`/`pg8000` for Postgres) and use `pandas.read_sql` to pull `plants`, `distribution_centers`, and `lane_costs` into DataFrames, then build the `plants`, `dcs`, and `cost` Python structures from those DataFrames — follow the pattern in Lecture 2 Section 4.

2. **Build the model.** One variable per `(plant, dc)` lane, `≤` supply constraints, `≥` demand constraints, objective = total shipping cost. Do **not** copy Lecture 2's exact code verbatim — build it from your own DataFrame-driven loop so you're sure you understand every line.

3. **Solve and print the shipping plan.** For every lane with `value() > 0`, print the plant, DC, and quantity. Print the total cost.

4. **Pull the shadow prices.** Loop over `prob.constraints.items()` and print each constraint's name, `.pi` (shadow price), and `.slack`. Identify which supply constraints have zero slack (fully used) and which have positive slack (spare capacity).

5. **Answer in a comment or a short `notes.md`:**
   - Every plant is fully used at the optimum (zero slack on all three supply constraints) — so which plant's supply constraint still shows the **largest-magnitude** shadow price, and which one shows **zero** despite also being fully used? What does that contrast mean in plain English?
   - Ho Chi Minh's supply constraint is binding but its shadow price should come back at (or essentially at) zero. Does that mean cutting Ho Chi Minh's capacity by 1,000 units would be free? Re-solve with that cut applied and find out — don't just reason from the shadow price alone.

## Expected result

- Optimal total cost: **$74,200**.
- Shipping plan: El Paso → Austin East (200 units), El Paso → Memphis (4,000 units), Guadalajara → Austin East (5,800 units), Ho Chi Minh → Memphis (1,000 units), Ho Chi Minh → Reno (4,000 units). Every other lane carries zero flow.
- All three supply constraints are binding (zero slack — every unit of capacity is used somewhere), but their shadow prices are **not** all equal: Guadalajara's should be the largest in magnitude at **$3** (it's the cheapest source into the network's most attractively-priced lane), El Paso's should be smaller but still nonzero at **$2**, and **Ho Chi Minh's should come back at exactly $0** even though its capacity is fully committed — a good check that "binding" and "valuable at the margin" are not the same thing.
- When you actually cut Ho Chi Minh's capacity by 1,000 units and re-solve: total supply (14,000) no longer covers total demand (15,000), so the model as originally written goes **infeasible** — you'll need Lecture 2 Section 6's dummy-source trick to get a solvable model back. That infeasibility, not a small cost bump, is the real answer to "was it free?"

## Done when…

- [ ] `solution.py` builds the model entirely from SQL-loaded DataFrames — no hard-coded plant/DC/cost dicts.
- [ ] Solved shipping plan matches the expected result above (same lanes, same quantities).
- [ ] Shadow prices are printed for every constraint, with slack.
- [ ] Your written answer correctly identifies which plant's shadow price is largest, which is zero, and correctly predicts (then confirms by re-solving) that Ho Chi Minh's zero shadow price does not mean its capacity is dispensable.

## Stretch

- Rewrite Task 2 using `scipy.optimize.linprog` instead of PuLP — you'll need to flatten the 9 `(plant, dc)` variables into a single vector and build the `A_ub`/`b_ub` matrices by hand. Confirm you get the same $74,200 total. (This is a good way to feel exactly why PuLP's algebraic style is easier for models like this.)
- Add a fourth "dummy" plant with 0 capacity and 0 cost everywhere, re-solve, and confirm the answer is completely unchanged — a zero-capacity dummy should never be used and should never distort the optimal plan. This is a useful pre-flight check before you build the "unbalanced network" pattern from Lecture 2 Section 6 for real.

## Submission

Commit `solution.py` (and `notes.md` if you used one) to your portfolio under `c40-week-09/exercise-02/`.
