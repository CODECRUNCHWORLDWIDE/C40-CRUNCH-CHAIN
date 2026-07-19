# Week 9 Homework — Optimization Practice Set

Extra reps, spaced across the week, tying together formulation, PuLP/SciPy mechanics, and interpretation. Do these after the matching lecture, not all at once on Sunday — formulation skill needs spacing, not cramming.

**Estimated time:** 5 hours across the week.

---

## Part A — Formulation drills (after Lecture 1, ~1.5h)

For each scenario below, do **not** write any code. Write out, on paper or in a text file:

1. The decision variables, named and with units.
2. The objective function (state clearly whether you're maximizing or minimizing).
3. Every constraint, in both words and symbols.

**A1. Blending.** Crunch Gear's fabric mill blends two fibers — recycled polyester (cheap, less durable) and virgin nylon (expensive, more durable) — into ripstop fabric. Each yard of fabric must be at least 40% nylon by weight to meet the durability spec. Recycled poly costs $2/lb, virgin nylon costs $5/lb, and each yard of fabric requires exactly 0.3 lb of fiber total. Minimize fiber cost per yard while meeting the durability spec. *(Hint: your decision variables are pounds of each fiber per yard, and "at least 40%" becomes a ratio constraint — clear the fraction before you write it as a linear inequality.)*

**A2. Staffing.** A Crunch Gear customer-support shift needs at least 8 agents during the 9am–1pm peak and at least 5 agents during the 1pm–6pm off-peak. Agents work one of two shift patterns: "Early" (9am–3pm, covers peak fully, covers 2 hours of off-peak) or "Late" (11am–6pm, covers 2 hours of peak, covers off-peak fully). Early shift costs $180, Late shift costs $160. Minimize total labor cost while covering both windows.

**A3. Diet-style blending.** A warehouse packing-material mix needs at least 200 lbs of cushioning material per shipment batch, made from air pillows ($0.10/lb, provides 1 unit of cushioning per lb) and shredded paper ($0.04/lb, provides 0.6 units of cushioning per lb). Minimize material cost while meeting the cushioning requirement. *(This is the classic "diet problem" shape — the oldest LP application there is, originally used to plan the cheapest nutritionally-adequate diet.)*

For each, also note: is this a maximization or minimization? Which constraints are "at least" (≥) and which are "at most" (≤)? Getting the direction of each inequality right, before any code, is most of the work.

---

## Part B — Solve what you formulated (after Lecture 1, ~1h)

Take **A2 (Staffing)** from Part A and solve it in PuLP. Report the optimal number of Early and Late shifts and the minimum total labor cost. Then answer: is the peak-window constraint or the off-peak-window constraint binding at the optimum — or both?

---

## Part C — Transportation variations (after Lecture 2, ~1.5h)

Using the Week 9 seed network (same plants, DCs, and lane costs from the [week README](./README.md)):

**C1.** Suppose Crunch Gear negotiates a **volume discount**: any lane carrying more than 3,000 units in a month gets a flat $0.50/unit discount on the *entire* lane's shipments (not just the units above 3,000). Model this as **two variables per lane** — one capped at 3,000 units at the normal rate, one uncapped at the discounted rate — and re-solve. Does the optimal total cost drop, and if so, by how much? *(This "convert a volume-tier into two linear pieces" trick is a standard way to keep a genuinely nonlinear discount inside a linear model — it only works because the discount rate change happens at a known breakpoint, not continuously.)*

**C2.** What is the maximum you could remove from Guadalajara's capacity before the optimal shipping plan is forced to route through a more expensive lane? Answer this two ways: (a) reason about it from the shadow price and the reduced costs of the nearest-alternative lanes, then (b) confirm by actually cutting the capacity in steps and re-solving until the plan changes. Report where your two answers agree or diverge.

---

## Part D — Facility location variation (after Lecture 3, ~1h)

Using the Lecture 3 facility-location scenario (four candidate DCs, three demand regions):

**D1.** Crunch Gear's board sets a rule: **at most 3** of the 4 candidate sites may be open in any plan (a headcount/systems-overhead limit, independent of the cost model). Add the constraint $\sum_k y_k \le 3$ and re-solve. Does the optimal set of open sites change from Lecture 3's answer? Since the unconstrained optimum already used exactly 3 sites, what does that tell you about whether this new rule was ever going to bind?

**D2.** Now flip it: the board instead requires **all four** candidates stay open (perhaps for redundancy reasons unrelated to cost). Force every $y_k = 1$ and re-solve — this collapses back to a pure transportation problem, since there's no longer any decision left in the `y` variables. Report the cost penalty of "open everything" versus the true unconstrained optimum from Lecture 3. This is the dollar cost of a redundancy policy — a number worth having whenever "keep everything open just in case" comes up in a real budget conversation.

---

## Submission

Commit a single `homework.md` (your written answers to Parts A and D) plus `part_b.py` and `part_c.py` (your PuLP solutions) to your portfolio under `c40-week-09/homework/`.
