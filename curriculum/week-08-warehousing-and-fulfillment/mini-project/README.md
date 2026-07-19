# Mini-Project — Slot & Batch Austin East DC

> Take Austin East DC's real one-month order-line pick profile, classify it, re-slot the warehouse, design a batching plan, and report exactly how many feet of pick travel and how many labor-hours the combined fix saves — all computed on SQL data, in pandas. No spreadsheets.

**Estimated time:** 2.5 hours, best done Saturday after the exercises and challenges.

This is the week's capstone, and it's the actual job. Nobody at Crunch Gear is going to hand you an isolated "what class is SKU 7" question — they're going to hand you a warehouse and a budget conversation, and expect a slotting plan, a batching recommendation, and a number: "here's what the current setup costs us in labor, here's what fixing it saves, here's what it would take to hit next quarter's peak-day volume."

---

## Deliverable

A directory in your portfolio `c40-week-08/mini-project/` containing:

1. `seed.sql` — the three tables (`warehouse_skus`, `pick_lines`, `warehouse_slots`) from the week README (copy them in, so this project is self-contained).
2. `analysis.py` — one script that reads all three tables from your database and produces every required output below.
3. `abc_classification.csv` — the ABC analysis output (see schema below).
4. `slotting_plan.csv` — the proposed re-slot (see schema below).
5. `report.md` — the written summary (see required sections below).
6. `notes.md` — a short reflection (see the end).

---

## Part 1 — ABC classification (`abc_classification.csv`)

For all **20 SKUs**, compute and include:

| Column | Meaning |
|---|---|
| `sku_id`, `sku_name`, `category` | identifiers |
| `pick_line_count` | total order lines this month (Exercise 1) |
| `units_picked` | total units this month |
| `pct_of_total` | this SKU's share of total pick lines |
| `cumulative_pct` | running cumulative share, sorted by `pick_line_count` descending |
| `abc_class` | `'A'` / `'B'` / `'C'`, per Lecture 2's cutoffs |

## Part 2 — the slotting plan (`slotting_plan.csv`)

Design a full re-slot of all 24 slots (20 assigned, 4 reserve slots left open for growth, exactly like the current layout). For each assigned slot, include:

| Column | Meaning |
|---|---|
| `slot_id`, `zone`, `distance_ft` | from `warehouse_slots` |
| `sku_id`, `sku_name` | the SKU you're assigning to this slot |
| `abc_class` | that SKU's class from Part 1 |
| `pick_line_count` | that SKU's monthly pick frequency |

**You must address the replenishment constraint from Challenge 1** (8 replenishment trips/month max into the golden zone) — a plan that ignores it and simply slots the top 6 SKUs by velocity into golden is not acceptable for this deliverable. If you didn't finish Challenge 1, work through its constraint here; the mini-project needs a *feasible*, not just theoretically-optimal, plan.

## Part 3 — `report.md`, required sections

### 1. Executive summary (≤150 words)

The current (alphabetical) slotting's total monthly pick-travel cost, your proposed re-slot's cost, and the percentage reduction — plus the one sentence that matters most to a manager: how many labor-hours per month does this free up.

### 2. ABC analysis summary

Render `abc_classification.csv`'s class boundaries as a small table (class, SKU count, cumulative % range) and name the single highest- and lowest-velocity SKU.

### 3. Slotting plan and the replenishment trade-off

Render `slotting_plan.csv`'s golden-zone rows. State the total replenishment trips your golden zone requires (must be ≤ 8) and explicitly name any SKU you deliberately kept **out** of the golden zone despite high velocity, and why — this is Challenge 1's finding, written up as a recommendation.

### 4. Batching plan

Using your slotting plan's distances, compute total pick-travel for the full month under **discrete** picking (one trip per line) and under **batched, one-wave-per-day** picking (Exercise 3's method). Report both totals and the percentage reduction from batching alone.

### 5. Combined impact

Report the single number that matters most: total monthly pick-travel-plus-touch labor time going from **(a)** the current alphabetical slotting with discrete picking, to **(b)** your re-slot with batched picking. Report feet, minutes, and the percentage reduction. Show your touch-time assumption (0.20 min/line, per Lecture 3) explicitly.

### 6. Peak-day staffing

Using your **re-slotted, batched** pick rate (minutes per line, computed from this project's own data — not copied from Challenge 2), size a picking crew for a **hypothetical peak day of 3,000 lines**, at 80% target utilization, on an 8-hour shift with a 30-minute break. Show the arithmetic (Lecture 3, Section 5's formula) and state the picker headcount.

---

## Milestones

- **Milestone 1 (30 min):** Seed the database, reproduce the ABC classification, write `abc_classification.csv`.
- **Milestone 2 (45 min):** Design the constrained slotting plan (Challenge 1's replenishment cap), write `slotting_plan.csv`, and confirm total golden-zone trips ≤ 8.
- **Milestone 3 (45 min):** Compute current-vs-proposed pick-travel distance (Part 3, Section 1) and the batching comparison (Part 3, Section 4).
- **Milestone 4 (30 min):** Combined-impact number (Section 5), peak-day staffing (Section 6), `report.md`, and `notes.md`.

---

## Rules

- **SQL + pandas only.** All three tables live in Postgres or SQLite; `analysis.py` reads them with `pd.read_sql` (or `sqlite3` + `pd.read_sql_query`) and does every computation in pandas/numpy. No spreadsheet touches this data at any point.
- **The slotting plan must be feasible**, not just theoretically optimal — it has to respect the 8-trip golden-zone replenishment cap from Challenge 1.
- **Every number in `report.md` needs its formula traceable** — a reviewer should be able to look at any figure and know which lecture's method produced it.
- **Round for display, not for calculation** — carry full precision through pandas, round only when writing the final report.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| ABC classification correctness | 20% | All 20 SKUs correctly classed, cumulative-% math matches the exercise-verified formulas |
| Slotting plan feasibility & quality | 30% | Golden zone respects the 8-trip cap; the plan minimizes pick-travel subject to that constraint, not just by velocity alone |
| Batching analysis | 15% | Discrete-vs-batched comparison correctly computed for the full month |
| Combined-impact number | 15% | (a)→(b) comparison correctly stacks the slotting and batching savings without double-counting |
| Peak-day staffing | 10% | Correct application of Lecture 3's labor-sizing formula to this project's own measured pick rate |
| Communication | 10% | `report.md` reads like something you'd send a warehouse operations manager — clear numbers, clear recommendation |

---

## Reflection (`notes.md`, ~200 words)

1. How much of the total labor-time savings came from slotting versus from batching? Which lever, on its own, would you prioritize first if you could only do one this quarter, and why?
2. Which SKU's placement decision was hardest to make, and why (tie it back to the velocity-vs-replenishment trade-off from Challenge 1)?
3. This project used a **one-dimensional** warehouse layout as a teaching simplification. What's the first thing that would break if you tried to apply this exact slotting method to a real two-dimensional warehouse with multiple aisles branching off a central corridor?
4. If Crunch Gear opened a second, larger DC next year, what's the one piece of this analysis you'd want *automated* (running on a schedule against live data) rather than redone by hand each quarter — and why that piece specifically?

---

## Why this matters

Every warehousing job comes down to exactly this exercise, at a bigger scale: take a real pick profile, apply consistent formulas, produce a slotting plan and a labor number, and be ready to defend both the mechanics and the trade-offs when someone asks "what if volume doubles?" Do this once carefully on a 20-SKU, one-month sample and the same script — parameterized, checked into version control — is what a real fulfillment operations team would run against a live warehouse-management-system feed next quarter.

When done: push, then take the [quiz](../quiz.md) and start Week 9 — Network Design & Facility Location.
