# Mini-Project — Route Plan + Mode/Carrier Plan for Crunch Gear

> Solve a lowest-cost delivery route for the Austin East local network, then build a mode-and-carrier plan for ten open orders — and defend the whole package on cost **and** service in a report a logistics manager would actually sign off on.

**Estimated time:** 2.5–3 hours, best done Saturday after the exercises and challenges.

This is the week's capstone, and it's deliberately built from the two halves of the job: **routing** (Lecture 2 — how do trucks move once they're loaded) and **mode/carrier selection** (Lectures 1 and 3 — what should carry the freight and who should drive it). A real logistics analyst does both, usually in the same week, usually for the same stakeholder meeting.

---

## Deliverable

A directory in your portfolio `c40-week-07/mini-project/` containing:

1. `route_plan.py` — your best routing solution for the 12-stop Austin East network (reuse or extend your Challenge 1 savings-algorithm code).
2. `mode_plan.sql` (and/or `.py`) — mode and carrier assignment for all 10 orders in `open_orders` (below).
3. `report.md` — the numbers and the reasoning, structured as described in Part 3.
4. `notes.md` — a short reflection (see the end).

Everything runs against `delivery_stops`, `shipments`, and the new `open_orders` table (seeded below) from PostgreSQL or SQLite. Note which engine you used.

---

## Part 1 — Best route plan (60–75 min)

Using `delivery_stops` (the same 12-stop, 120-case-truck network from Lecture 2 and Exercises), produce your **best available route plan**:

1. Run your Challenge 1 savings-algorithm implementation (or, if you skipped Challenge 1, your Exercise 2 capacitated nearest-neighbor as a fallback — but note in `report.md` that you did).
2. Report: number of trucks used, total distance, and each route's stop sequence and load.
3. Compute **average capacity utilization** across your routes (`avg(load / 120)`). Is there a route running well under capacity that could plausibly absorb a stop from an overloaded neighbor to balance the fleet better? You don't have to re-optimize by hand — just identify it and say so.

## Part 2 — Mode + carrier plan for 10 open orders (75–90 min)

Seed the new order slate:

```sql
CREATE TABLE open_orders (
    order_id          INTEGER PRIMARY KEY,
    description        TEXT    NOT NULL,
    lane_id             TEXT    NOT NULL,
    weight_lbs          NUMERIC NOT NULL,
    deadline_days        NUMERIC NOT NULL,
    stockout_cost_usd    NUMERIC NOT NULL
);

INSERT INTO open_orders VALUES
(1,  'Regular wholesale reorder',            'AUS-DAL', 3200,  4,   700),
(2,  'DTC rush order, VIP customer',          'AUS-HOU',  900,  2,  5200),
(3,  'DC replenishment, low urgency',         'MEM-CHI', 26000, 5,   250),
(4,  'Wholesale reorder, Pacific NW account', 'REN-SEA', 1100,  3,  1800),
(5,  'Wholesale reorder, standard',           'AUS-PHO', 7800,  4,   950),
(6,  'Small boutique rush order',             'MEM-NEW',  300,  2,  3600),
(7,  'Routine fabric import, no urgency',     'HPH-ATX', 20000, 40,  500),
(8,  'Regional DC top-off, tight window',      'REN-DEN', 12500, 3,  2200),
(9,  'Trade-show samples, hard date',          'AUS-ATL',   60,  2,  1500),
(10, 'DC-to-DC replenishment, flexible',       'MEM-AUS', 5000,  6,   300);
```

For each of the 10 orders:

1. Using the mode `cost_per_lb` figures and typical transit-day ranges from Lecture 1 / Exercise 1, **shortlist 2-3 feasible modes** (feasible = plausibly meets `deadline_days`).
2. Using the carrier scorecard from Exercise 3 (carriers with `n_shipments >= 10` only — don't recommend a carrier you don't have enough data on), **pick a specific carrier** for the mode you land on, where that carrier already operates on the matching lane or mode in the `shipments` history. If no qualifying carrier has history on that exact lane, pick one with strong history on that **mode**, and say so.
3. Write the one-sentence justification, same rule as Challenge 2: tie it to `deadline_days` and `stockout_cost_usd`, not just "cheapest" or "fastest" in isolation.

## Part 3 — The report (`report.md`)

Structure it in three sections:

1. **Route plan summary** — trucks used, total miles, utilization, and one sentence comparing your result to the nearest-neighbor baseline (345.7 mi / 4 trucks) with a percentage improvement.
2. **Order-by-order mode/carrier table** — all 10 orders, one row each, columns: mode chosen, carrier chosen, estimated cost, justification sentence.
3. **Portfolio-level summary** — total estimated freight cost across all 10 orders, compared to the two naive baselines from Challenge 2 (cheapest-mode-always, Air-always). State in one paragraph whether your plan is closer to the cheap end or the fast end of that range, and why that's the right place for Crunch Gear to sit this week specifically (look at how many orders have a high `stockout_cost_usd` relative to their freight cost — that ratio should be steering your overall posture).

---

## Milestones

- **Milestone 1 (60–75 min):** Part 1 — routing plan and utilization check.
- **Milestone 2 (75–90 min):** Part 2 — mode and carrier plan for all 10 orders.
- **Milestone 3 (30 min):** Part 3 — assemble `report.md`, write `notes.md`.

---

## Rules

- **Route plan must be capacity-valid** — no truck over 120 cases, every stop visited exactly once. Re-run your Challenge 1 validation assertions here too.
- **Every carrier recommendation must come from a carrier with `n_shipments >= 10`** in the historical `shipments` data, per the Exercise 3 confidence rule. If you genuinely think a low-volume carrier (Pacific Rim Ocean Lines, SkyBridge Air Cargo) is still the right call for a specific order, you may recommend them — but you must say explicitly that you're doing so with low confidence and why the situation justifies it anyway.
- **No naked numbers.** Every mode/carrier choice needs its one-sentence justification in the report table, not just in your head.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Route plan correctness | 20% | Capacity-valid, assertions pass, beats or matches the NN baseline |
| Mode/carrier correctness | 25% | Every choice is feasible against its deadline; every carrier has adequate history |
| Justification quality | 25% | Each of the 10 justifications ties explicitly to deadline and stockout cost, not generic reasoning |
| Portfolio-level synthesis | 20% | `report.md` Part 3 makes a real, numbers-backed argument about where the plan sits on the cost/speed spectrum |
| Clarity | 10% | Report reads like something you'd actually hand to a manager — tables, not walls of text |

---

## Reflection (`notes.md`, ~200 words)

1. Which of the 10 orders was the hardest mode/carrier call, and why?
2. Where did the routing side and the mode/carrier side of this project interact — did anything about the route plan change how you thought about the orders, or vice versa?
3. If Crunch Gear gave you a bigger delivery fleet (say, 6 trucks instead of "however many the savings algorithm needs"), what would you do differently, and would it actually help?
4. One thing you'd want to know about a carrier that isn't in this week's data (e.g., damage/claims rate, driver turnover, capacity availability during peak season) — and why it would change a real decision.

---

## Why this matters

This mini-project is the two halves of a logistics analyst's actual job, back to back: get the truck to visit its stops efficiently, and get the right freight on the right carrier before it ever leaves the dock. Do both with numbers instead of habit, and you've done in an afternoon what a lot of freight teams still do by gut feel. Keep this project — Week 8 (warehousing) picks up right where the truck stops, at the dock door.

When done: push, then take the [quiz](../quiz.md).
