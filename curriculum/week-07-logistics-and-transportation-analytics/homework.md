# Week 7 — Homework

Five problems, ~5 hours total, spread across the week. These reinforce the lectures with a mix of hands-on SQL, a Python routing extension, and two written-reasoning tasks. Commit each.

All SQL runs against `shipments` and `delivery_stops` from the [README](./README.md) unless a problem says otherwise.

---

## Problem 1 — Twenty warm-up queries (75 min)

Write and run each against `shipments`. Put them in `warmups.sql` with a `-- N` comment and the answer beneath each.

1. Total number of shipments per mode.
2. The single most expensive shipment (by `freight_cost`) and its lane.
3. Every shipment on the `MEM-CHI` lane, sorted by `ship_date`.
4. Average `weight_lbs` per mode.
5. The three carriers with the highest total spend.
6. Every shipment where `actual_transit_days > promised_transit_days` (late shipments), count them.
7. Total freight spend in January 2025 vs. February 2025 (two queries, or one with `GROUP BY` on the month).
8. The lane with the highest total weight shipped.
9. Every `Parcel` shipment with `weight_lbs > 30` (the heavier end of parcel).
10. Distinct `origin` values (should be 4: three DCs plus the Hai Phong supplier).
11. The carrier with the *fewest* shipments overall.
12. Total units shipped, all modes combined.
13. Every shipment carried by `Longhaul Truckload`, sorted by `freight_cost` descending.
14. The average `freight_cost` per shipment, `FTL` only.
15. Every lane that has **both** a `Parcel` and an `FTL` shipment (hint: `GROUP BY lane_id HAVING COUNT(DISTINCT mode) ...`).
16. The single cheapest shipment by `freight_cost` (not per pound — raw cost).
17. Total freight cost for shipments originating at each DC (`Austin East`, `Memphis DC`, `Reno DC`) — exclude the Hai Phong import lane.
18. Every shipment that shipped on a Monday. *(Hint: Postgres `EXTRACT(DOW FROM ship_date)`, SQLite `strftime('%w', ship_date)` — Sunday is 0 in both.)*
19. The mode with the highest **variance** in `freight_cost` (eyeball it via `MAX - MIN`, or use `STDDEV`/`VARIANCE` if your engine supports it).
20. Every shipment where `weight_lbs` is within 5% of a "round" break point (500, 1000, 2000, 5000, 10000, 20000) — a candidate list for the "should this have been rounded up or down" conversation from Lecture 1.

---

## Problem 2 — Extend the delivery network (60 min)

Make the routing network your own.

1. Write `INSERT` statements adding **3 new stops** to `delivery_stops` (invent names, coordinates, and demand — keep demand between 15 and 60 cases so the network stays realistic).
2. Re-run your Exercise 2 capacitated nearest-neighbor (120-case capacity) on the new 15-stop network (depot + 15 customers). How many trucks does it need now, and what's the new total distance?
3. Re-run your Challenge 1 savings algorithm (if you completed it) on the same 15-stop network. Does it still beat nearest-neighbor, and by roughly the same percentage as before, or does the margin change?
4. **Deliver** `extend_network.py` (or `.sql` + `.py`) with the inserts, both routing runs, and 3-4 sentences on what changed and why.

---

## Problem 3 — Explain the weight-break trap (30 min)

In `weight-break-writeup.md`, answer in prose (no more than 350 words total):

1. In your own words, why can a shipment that weighs *more* sometimes cost *less* than one that weighs slightly less, on the same LTL lane?
2. Find one real example in the `shipments` table where two shipments on the **same lane and mode** have `freight_cost / weight_lbs` differing by more than 30%. List both `shipment_id`s and their cost-per-lb. What's the most likely explanation — different carriers, different weight breaks, or something else? State your reasoning.
3. If you were advising Crunch Gear's planning team, what's one operational rule (about order timing, order batching, or minimum shipment size) you'd suggest to reduce this kind of cost variance?

---

## Problem 4 — Carrier scorecard, one query (45 min)

In `scorecard.sql`, write a **single query** (one `SELECT`, CTEs allowed) that returns, for every carrier with at least 8 shipments: `carrier`, `n_shipments`, `on_time_pct`, `total_spend`, `avg_cost_per_lb`, and a `rank` column (using `RANK()` or `ROW_NUMBER()`) ordering carriers best-to-worst by `on_time_pct` with `avg_cost_per_lb` as the tiebreaker.

**Deliver** the query plus 2-3 sentences: which carrier ranks #1, and would you recommend Crunch Gear route *more* freight to them, less to the bottom-ranked carrier, or is there not enough information in this table alone to make that call? (There usually isn't — say what else you'd want to know: damage rates, capacity availability, rate stability.)

---

## Problem 5 — A routing "what if" (60 min)

Crunch Gear is considering opening a **second depot** in San Antonio to shorten the southern routes (San Antonio North, San Antonio South, New Braunfels, San Marcos currently require the longest legs from Austin East).

1. Using the same (x, y) coordinate approach as `delivery_stops`, place a hypothetical San Antonio depot at roughly `(0, -45)` (near the San Antonio stops).
2. Reassign the 4 southernmost stops (San Marcos River Gear, New Braunfels Alpine Traders, San Antonio North Summit Sports, San Antonio South Trailblazers) to the new depot, and run nearest-neighbor or savings on just those 4 stops from the new depot.
3. Compare: total distance for those 4 stops served from Austin East (part of your Exercise 2 / Challenge 1 routes) vs. served from the new San Antonio depot.

**Deliver** `second-depot.py` with both scenarios computed, plus 3-4 sentences on whether the mileage savings alone would justify standing up a second depot (mention at least one cost a second depot brings that pure routing distance doesn't capture — rent, staffing, duplicate inventory, etc.).

---

## Time budget

| Problem | Time |
|--------:|----:|
| 1 | 75 min |
| 2 | 60 min |
| 3 | 30 min |
| 4 | 45 min |
| 5 | 60 min |
| **Total** | **~4.5 h** |

After homework, take the [quiz](./quiz.md) and ship the [mini-project](./mini-project/README.md).
