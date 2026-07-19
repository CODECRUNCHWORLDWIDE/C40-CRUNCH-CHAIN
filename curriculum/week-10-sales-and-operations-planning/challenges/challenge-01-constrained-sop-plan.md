# Challenge 1 — Build a Constrained S&OP Plan

**Time:** ~2 hours. **Difficulty:** Hard. **No single numeric answer key.**

## The scenario

Exercise 3 proved a hard fact: closing each month's gap using only *that month's own* overtime and subcontract capacity is **not enough** — April still stocks out by 650 units even maxed out, because there wasn't enough buffer carried in from earlier. Crunch Gear's VP of Operations has asked you for the actual six-month production plan: exactly how many units of regular time, overtime, and subcontract to run **in every month, Trail Footwear only**, such that the family **never drops below its 900-unit safety-stock target, in any month, Jan–Jun** — and to do it as cheaply as possible.

This is a genuine constrained-optimization problem. You already have every number you need: `demand_plan`, `supply_plan` (with the monthly ceilings and costs for regular/overtime/subcontract), and `product_reference` (beginning inventory, safety-stock target). What you don't have is a formula that solves it for you — that's the challenge.

## Your task

1. **Prove feasibility exists before you optimize.** First, confirm a *some* plan can work at all: what's the largest single-month deficit (relative to the 900-unit floor) across the regular-capacity-only balance table from Exercise 1? That number is the minimum total extra units (overtime + subcontract, combined, anywhere across the horizon) your plan needs to close, at minimum.

2. **Build a full six-month plan.** For every month, decide: how many overtime units to use (0 up to that month's `overtime_capacity_units`), and how many subcontract units to use (0 up to that month's `subcontract_capacity_units`). Compute the resulting running ending inventory for all six months and confirm it **never drops below 900.**

3. **Minimize the added cost.** Overtime and subcontract cost different amounts in different months (`unit_cost_overtime` and `unit_cost_subcontract` in `supply_plan` — they rise slightly from Q1 to Q2). A plan that uses *every available unit of every lever in every month* is guaranteed feasible, but almost certainly **not** the cheapest feasible plan. Think about which lever, in which month, is the cheapest way to close the total gap from Step 1 — and whether it matters *when* in the six months you use a given lever, given that capacity used in an earlier month helps every month after it, but capacity used in a later month only helps from that month onward.

4. **Report the total cost.** Sum the incremental cost (beyond what regular-capacity-only production would have cost) of every overtime and subcontract unit your plan uses. Compare it explicitly to the cost of the "use every available unit, every month" plan (also compute that one — it's a useful, easy-to-build baseline even though you shouldn't submit it as your final answer).

## Constraints

- Trail Footwear only. Do not touch Backpacks & Bags or Apparel (Exercise 3 already showed they need no help this cycle).
- Regular capacity is **always fully used first** in every month — you're never turning down regular capacity for a cheaper lever, since regular time is always the cheapest unit cost of the three.
- Ending inventory must be `>= 900` (the safety-stock target) at the end of **every** month, not just by June.
- You may not exceed any month's stated `overtime_capacity_units` or `subcontract_capacity_units` ceiling.

## Hints

<details>
<summary>On Step 1 — the minimum total extra needed</summary>

Compare the regular-capacity-only ending-inventory sequence from Exercise 1 (`1800, 1700, 800, -2500, -5700, -7700`) against the 900-unit floor at each point. Because a unit of extra capacity added in an early month raises *every subsequent month's* running balance by the same amount (it never expires or gets "used up" by an earlier month), the single largest deficit anywhere in the sequence tells you the true minimum total you need — you don't need to separately solve each month's deficit as an independent problem.

</details>

<details>
<summary>On Step 3 — ordering the levers by cost</summary>

Regular capacity's unit cost never changes this challenge — it's already committed. Compare `unit_cost_overtime` and `unit_cost_subcontract` month by month: is overtime cheaper than subcontract everywhere, or does the ordering ever flip? Within overtime alone, does every month cost the same, or does it step up partway through the horizon? A cost-minimizing plan reaches for the cheapest available lever first, everywhere it can, before reaching for a more expensive one — but only up to how much of that lever is actually available each month.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Feasibility | Asserts the plan works | Shows a table with all 6 months' ending inventory, every one `>= 900` |
| Constraint respect | Exceeds a month's overtime/subcontract ceiling somewhere | Every month's usage is checked against its stated ceiling |
| Cost discipline | Just maxes every lever, every month | Computes and beats the "max everything" baseline, with the savings stated in dollars |
| Reasoning | No explanation of lever choice | States which lever was preferred in which months, and why, tied to the actual cost numbers |

## Submission

Commit `challenge-01.sql` (or `.sql` + `.py`) with your plan, the verification table proving it never breaches the floor, and your total incremental cost (with the "max everything" baseline cost alongside it for comparison) to your portfolio under `c40-week-10/challenge-01/`.
