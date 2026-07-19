# Lecture 2 — Slotting & Pick-Path Optimization

> **Duration:** ~2 hours. **Outcome:** You can run an ABC velocity analysis on an order-line pick profile in SQL, design a slotting plan from the result, and compute exactly how many feet (and minutes) of pick travel a slotting plan produces — current or proposed.

Lecture 1 established the claim: travel is 50–70% of pick time, and where you put things determines how much of it you pay. This lecture makes that claim computable. By the end you will have re-slotted Austin East DC and be able to state, in a single sentence with a number attached, how much travel the fix saves.

## 1. ABC analysis — the Pareto principle applied to a warehouse

**ABC analysis** ranks SKUs by some measure of importance — here, **pick frequency** (how many order lines call for that SKU) — and buckets them into three tiers based on their **cumulative share** of total activity:

- **Class A** — the SKUs responsible for roughly the first 70–80% of cumulative pick activity. Usually a *minority* of SKUs.
- **Class B** — the next slice, out to roughly 90–95% cumulative.
- **Class C** — the long tail: many SKUs, each individually rare, together making up the last 5–10%.

This is the same **80/20 rule** (Pareto principle) you'll meet in almost every operations discipline — a small share of causes drives a large share of the effect. In inventory management it shows up as "20% of SKUs drive 80% of revenue"; in warehousing it shows up as "a minority of SKUs drive the overwhelming majority of picks," and that's precisely the fact a good slotting plan exploits.

**Important: there's no law that the cutoff lands exactly on 20%/30%/50% of your SKU count.** That 80/20 phrasing is a memorable rule of thumb, not a guarantee. The *real* rule is: **compute the actual cumulative percentage from your actual data, and cut where the data says to cut.** You'll see exactly that below — Austin East's cutoff doesn't land on a round SKU count, and that's normal.

## 2. Computing pick frequency in SQL

The whole analysis starts from one `GROUP BY` against this week's `pick_lines` table:

```sql
SELECT
    sku_id,
    COUNT(*) AS pick_line_count
FROM pick_lines
GROUP BY sku_id
ORDER BY pick_line_count DESC;
```

```
 sku_id | pick_line_count
--------+-----------------
      1 |              55
      2 |              40
      3 |              30
      4 |              20
      5 |              10
      6 |               9
      7 |               8
      8 |               7
      9 |               6
     10 |               6
     11 |               5
     12 |               5
     13 |               4
     14 |               4
     15 |               4
     16 |               3
     17 |               3
     18 |               3
     19 |               2
     20 |               1
```

That's already useful on its own — SKU 1 (`Hydration Bladder 2L`) was picked 55 times this month, 55x more often than SKU 20 (`4-Season Mountaineering Tent`), picked once. But "picked a lot" isn't a classification yet. You need the **running (cumulative) share**, which means a window function.

## 3. Cumulative share with a window function

```sql
WITH freq AS (
    SELECT sku_id, COUNT(*) AS pick_line_count
    FROM pick_lines
    GROUP BY sku_id
),
ranked AS (
    SELECT
        sku_id,
        pick_line_count,
        SUM(pick_line_count) OVER (ORDER BY pick_line_count DESC
                                    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total,
        SUM(pick_line_count) OVER () AS grand_total
    FROM freq
)
SELECT
    sku_id,
    pick_line_count,
    ROUND(100.0 * pick_line_count / grand_total, 2)  AS pct_of_total,
    ROUND(100.0 * running_total  / grand_total, 2)   AS cumulative_pct
FROM ranked
ORDER BY pick_line_count DESC;
```

`SUM(...) OVER (ORDER BY pick_line_count DESC ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)` is a **running total** — for each row, it sums every row from the top of the ranking down to (and including) the current one. Pair that with the grand total (a window `SUM` with no `ORDER BY`, so it sums *every* row regardless of position) and you get a cumulative percentage in one pass, no self-join, no subquery-per-row.

Running this against Austin East's 225-line profile produces:

```
 sku_id | pick_line_count | pct_of_total | cumulative_pct
--------+------------------+--------------+----------------
      1 |               55 |        24.44 |          24.44
      2 |               40 |        17.78 |          42.22
      3 |               30 |        13.33 |          55.56
      4 |               20 |         8.89 |          64.44
      5 |               10 |         4.44 |          68.89
      6 |                9 |         4.00 |          72.89
      7 |                8 |         3.56 |          76.44
      8 |                7 |         3.11 |          79.56
      9 |                6 |         2.67 |          82.22
     10 |                6 |         2.67 |          84.89
     11 |                5 |         2.22 |          87.11
     12 |                5 |         2.22 |          89.33
     13 |                4 |         1.78 |          91.11
     14 |                4 |         1.78 |          92.89
     15 |                4 |         1.78 |          94.67
     16 |                3 |         1.33 |          96.00
     17 |                3 |         1.33 |          97.33
     18 |                3 |         1.33 |          98.67
     19 |                2 |         0.89 |          99.56
     20 |                1 |         0.44 |         100.00
```

## 4. Drawing the class boundaries

Apply the standard cutoffs — **A ≤ 80%, B 80–95%, C > 95%** — by reading straight down the `cumulative_pct` column:

| Class | SKUs | Cumulative range | SKU count | Share of SKU count |
|-------|------|-------------------|-----------:|----------------------:|
| **A** | 1–8 (`Hydration Bladder`, `Socks`, `Headlamp`, `Poncho`, `Water Bottle`, `Trekking Poles`, `Camp Mug`, `First Aid Kit`) | 0 – 79.56% | 8 | 40% |
| **B** | 9–15 (`Daypack`, `Dry Bag`, `Compression Sack`, `Runner Shoes M`, `Runner Shoes W`, `Cookset`, `Camp Chair`) | 79.56 – 94.67% | 7 | 35% |
| **C** | 16–20 (`Ultralight Tent`, `Sleeping Bag`, `Stove Kit`, `Expedition Backpack`, `Mountaineering Tent`) | 94.67 – 100% | 5 | 25% |

Notice the A cutoff falls **between SKU 8 (79.56%) and SKU 9 (82.22%)** — not on a clean round number, and not at "20% of the catalog." Class A here is 40% of the SKU count, not the textbook 20%. That's real data behaving like real data: apply the 80% rule literally to the cumulative column, don't force the SKU count to match a memorized ratio. (You'll compute this exact table yourself, from scratch, in Exercise 1 — this section shows the mechanics, not the full worked answer.)

**In pandas, the equivalent is three lines:**

```python
import pandas as pd

freq = df.groupby("sku_id").size().reset_index(name="pick_line_count")
freq = freq.sort_values("pick_line_count", ascending=False).reset_index(drop=True)
freq["cumulative_pct"] = 100 * freq["pick_line_count"].cumsum() / freq["pick_line_count"].sum()
```

`cumsum()` is pandas' running total — it's the exact pandas equivalent of the SQL window function above, and it's the one method you'll use over and over this week.

## 5. From ABC class to slotting zone

The whole point of the classification is the mapping that follows it:

| ABC class | Slotting zone | Why |
|-----------|-----------------|-----|
| **A** | **Golden** (closest to pack station) | Picked constantly — every foot of distance here is multiplied by a large trip count |
| **B** | **Middle** | Picked regularly, but the travel-cost multiplier is smaller — a farther walk hurts less |
| **C** | **Reserve** (farthest) | Picked rarely — a long walk once or twice a month barely moves the total |

Austin East DC's *current* slotting (from the [week setup](../README.md)) does almost the opposite: it was assigned alphabetically by SKU name, and the alphabet has no relationship whatsoever to pick frequency. Look at where two of the extremes actually sit today:

```sql
SELECT w.slot_id, w.zone, w.distance_ft, s.sku_id, s.sku_name
FROM warehouse_slots w
JOIN warehouse_skus s ON s.sku_id = w.current_sku_id
WHERE s.sku_id IN (1, 20)
ORDER BY s.sku_id;
```

```
 slot_id | zone   | distance_ft | sku_id | sku_name
---------+--------+--------------+--------+------------------------------
 M7      | Middle |           92 |      1 | Hydration Bladder 2L
 G2      | Golden |           24 |     20 | 4-Season Mountaineering Tent
```

`Hydration Bladder 2L` — Class A, 55 picks this month, the single busiest SKU in the building — sits 92 feet out. `4-Season Mountaineering Tent` — Class C, picked exactly **once** — occupies a prime 24-foot golden-zone slot. Every one of those 55 monthly trips to the bladder walks past three empty-ish golden slots to reach it. That's not a rounding error; it's the single biggest opportunity in this week's dataset, and it's sitting in a real table you already queried.

## 6. Computing total pick-travel distance

Once every SKU has a slot (current or proposed) and you know how many times each SKU was picked, total travel distance is one join and one arithmetic column away:

```sql
SELECT
    SUM(f.pick_line_count * 2 * w.distance_ft) AS total_travel_ft
FROM (
    SELECT sku_id, COUNT(*) AS pick_line_count
    FROM pick_lines
    GROUP BY sku_id
) f
JOIN warehouse_slots w ON w.current_sku_id = f.sku_id;
```

The `* 2` matters: `distance_ft` is **one-way**, and a pick is a **round trip** — the picker walks to the slot, then walks back toward the pack station with the item. Forget the `* 2` and every distance number in this lecture, and everything downstream of it, is half of reality.

Running this against Austin East's **current alphabetical slotting**:

```
 total_travel_ft
------------------
            44466
```

**44,466 feet** — that's **8.4 miles** of walking, generated by this one month's 225 picks, purely because of *where things are stored*. At a typical warehouse walking-and-searching pace of **250 ft/minute**, that's **177.9 minutes** — nearly three full labor-hours — spent walking, in one month, from one small DC's pick profile alone. Multiply that by every month of the year and by a real DC's actual order volume (thousands of lines a day, not 225 a month) and the size of the number should stop looking academic.

## 7. Re-slotting by velocity — the assignment principle

The optimal fix follows directly from ABC classification: **pair your highest-frequency SKUs with your shortest-distance slots, in matching rank order.** This is a classic assignment problem, and for a single scalar cost (distance × frequency) the optimal solution is exactly what intuition suggests: sort SKUs by pick frequency descending, sort slots by distance ascending, and pair them position-for-position.

```sql
WITH freq_ranked AS (
    SELECT sku_id, COUNT(*) AS pick_line_count,
           ROW_NUMBER() OVER (ORDER BY COUNT(*) DESC) AS freq_rank
    FROM pick_lines
    GROUP BY sku_id
),
slots_ranked AS (
    SELECT slot_id, zone, distance_ft,
           ROW_NUMBER() OVER (ORDER BY distance_ft ASC) AS dist_rank
    FROM warehouse_slots
    WHERE slot_id NOT LIKE 'R5' AND slot_id NOT LIKE 'R6'
      AND slot_id NOT LIKE 'R7' AND slot_id NOT LIKE 'R8'  -- exclude empty reserve slots
)
SELECT f.sku_id, f.pick_line_count, s.slot_id, s.zone, s.distance_ft
FROM freq_ranked f
JOIN slots_ranked s ON s.dist_rank = f.freq_rank
ORDER BY f.freq_rank;
```

Applying this re-slot and recomputing total travel distance with the same query from Section 6:

```
 total_travel_ft
------------------
            19284
```

**19,284 feet — down from 44,466.** That's a **56.6% reduction**, from 177.9 minutes of monthly pick travel to **77.1 minutes** — a savings of just over **100 minutes, 1.7 labor-hours, from re-slotting alone**, with zero new racking, zero new headcount, and zero new software. You'll reproduce this exact number yourself in Exercise 2, and Challenge 1 asks you to check whether it survives a real-world constraint this simple version glosses over: replenishment frequency (Lecture 1, Section 6).

## 8. Pick-path distance for a multi-line order

Section 6's model treats every pick line as an independent round trip — realistic for a **discrete (single-order) picking** operation where a picker carries one order's tote and walks to each of that order's lines in sequence, then returns. For a multi-line order, the *path* through several slots also matters, not just which zone each slot sits in.

Take `order_id 9` from this week's `pick_lines` table — five lines, five different SKUs:

```sql
SELECT sku_id, qty_picked FROM pick_lines WHERE order_id = 9 ORDER BY pick_line_id;
```

```
 sku_id | qty_picked
--------+------------
      3 |          2
      6 |          1
      4 |          3
      8 |          1
      1 |          1
```

If a picker visits these five slots in the order the order-lines happen to list them, they walk to whatever distance each slot sits at, in that arbitrary sequence — no better than random. If instead they sort their walk by distance ascending (nearest-first, a **greedy pick path**, the picking-floor cousin of Week 7's nearest-neighbor routing heuristic), they visit the closest slot first and work outward, which minimizes backtracking. On a **single line of aisles** (this week's simplified one-dimensional layout), sorting slots by `distance_ft` ascending and picking in that order **is** the shortest possible path — no heuristic needed, because there's only one way to walk out and back on a line. On a real two-dimensional warehouse floor this becomes a small traveling-salesman-style problem, exactly like Week 7's delivery routing, just walked instead of driven. Exercise 2 has you compute both the naive and the sorted pick path for a real multi-line order and show the difference.

## 9. What you now know how to do

- Rank SKUs by pick frequency and compute a running cumulative share, in SQL with a window function and in pandas with `cumsum()`.
- Apply the standard A (≤80%) / B (80–95%) / C (>95%) cutoffs to real data — and expect the cutoff to land on a real, not-round SKU count.
- Compute total pick-travel distance for any slotting plan with one join and one multiplication (frequency × 2 × distance).
- Re-slot a warehouse optimally for single-scalar travel cost by pairing sorted frequency against sorted distance.
- Recognize that within a single multi-line order, the *sequence* you visit slots in also matters, and sorting by distance minimizes it on a simple layout.

Lecture 3 builds on top of this fixed, re-slotted warehouse and asks the next question: if you batch several orders into one wave instead of picking them one at a time, how many fewer trips do you need — and how many pickers does that let you run with, at a stated order volume?
