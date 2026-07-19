# Week 5 — Homework

Five problems, ~5 hours total, spread across the week. These reinforce all three lectures with fresh numbers — not the seed catalog — so you can't pattern-match your way through without actually applying the formulas. Commit each.

---

## Problem 1 — EOQ on three new SKUs (60 min)

Crunch Gear is adding three SKUs to a different regional catalog. Compute `H`, `EOQ`, `orders_per_year`, and `TC(Q*)` for each, by hand first, then check yourself with a short script.

| SKU | Annual demand `D` | Order cost `S` | Unit cost `C` | Holding rate `i` |
|---|---:|---:|---:|---:|
| Rain Shell Poncho | 6,000 | $100 | $35.00 | 0.22 |
| Ultralight Tent 2P | 900 | $150 | $220.00 | 0.28 |
| Hydration Bladder 2L | 15,000 | $70 | $16.00 | 0.20 |

**Deliver** `problem-01.md` with your by-hand work (show `H`, then `EOQ`, then `TC(Q*)`) for all three, plus `problem-01.py` confirming each. *(Expect `EOQ` around 395, 66, and 810 units respectively — if you're off by more than a rounding error, recheck which numbers went inside vs. outside the square root.)*

---

## Problem 2 — Safety stock at three different targets (60 min)

Same three SKUs, now with lead-time data and a **different target service level for each** (deliberately — this is how a real catalog looks; not every SKU gets the same target).

| SKU | `σ_d` (daily) | `L` (days) | `σ_L` (days) | Target CSL |
|---|---:|---:|---:|---:|
| Rain Shell Poncho | 7.0 | 14 | 2.5 | 95% |
| Ultralight Tent 2P | 2.0 | 25 | 4.0 | 90% |
| Hydration Bladder 2L | 18.0 | 10 | 1.5 | 99% |

For each: compute `σ_DLT`, safety stock, and reorder point. **Deliver** `problem-02.md` with the work shown and one sentence per SKU on why you think Crunch Gear picked *that* target for *that* SKU (tent = expensive and slow-moving; bladder = cheap, high-volume, always-in-stock expectation; poncho = middle-of-the-road — argue it in your own words).

---

## Problem 3 — Pick the right policy (45 min)

For each situation below, state which policy — **(s,Q)**, **(R,S)**, or **base-stock (S-1,S)** — you'd recommend, and justify it in 2–3 sentences using Lecture 3's decision guide. There's a best answer for each, but the justification matters more than the label.

1. Crunch Gear's warehouse management system updates inventory counts in real time as units are picked and received.
2. A supplier only accepts purchase orders on the first Monday of each month, batched with other retailers on the same production run.
3. A $340 limited-run alpine ice axe sells roughly 2 units a week; Crunch Gear doesn't want to tie up capital in a large standing safety stock for such a low-volume, high-cost item.
4. Three accessory SKUs share a single supplier and get consolidated onto one truck every other Friday to hit a freight minimum.

**Deliver** `problem-03.md`.

---

## Problem 4 — Newsvendor with a pricing lever (60 min)

Crunch Gear is producing a **Founders Edition** anniversary hard-shell jacket, one-time run, no reorder possible.

| Input | Value |
|---|---|
| Selling price | $220 |
| Unit cost | $95 |
| Salvage value | $50 |
| Forecast demand | Normal, `μ` = 650, `σ` = 140 |

1. Compute `Cu`, `Co`, `CR`, `z`, and `Q*`.
2. Marketing is considering raising the price to **$250** (they believe demand won't drop meaningfully at this price point — assume the same `μ` and `σ`). Recompute `Cu`, `Co`, `CR`, `z`, and `Q*` at the new price.
3. In 3–4 sentences: does raising the price make Crunch Gear order *more* or *fewer* units, and does that match your intuition about what a higher margin should do to the optimal order quantity? Explain the mechanism (which cost went up, which stayed the same, and how that moved the critical ratio).

**Deliver** `problem-04.md` and `problem-04.py`. *(Expect `Q*` around 738 at $220 and around 756 at $250.)*

---

## Problem 5 — Extend the catalog (75 min)

Add **three SKUs of your own invention** to a copy of the Week 5 seed table — pick a product category Crunch Gear doesn't currently sell (sunglasses, camp cookware, gaiters, whatever you like), and invent plausible `unit_cost`, `annual_demand`, `order_cost`, `holding_pct`, `lead_time_days`, `demand_std_daily`, and `lead_time_std_days` for each.

1. Write the `INSERT` statements and load them (you now have 13 rows).
2. Compute the full policy — `EOQ`, `safety_stock` at 95% CSL, `reorder_point`, and `total_annual_policy_cost` — for your 3 new SKUs, using the same formulas as the mini-project.
3. Write 3–4 sentences: which of your invented SKUs turned out to be the most expensive to stock, and does that match what you expected when you invented its numbers? If not, what surprised you?

**Deliver** `problem-05.sql` (inserts) and `problem-05.md` (results + reflection).

---

## Time budget

| Problem | Time |
|--------:|-----:|
| 1 | 60 min |
| 2 | 60 min |
| 3 | 45 min |
| 4 | 60 min |
| 5 | 75 min |
| **Total** | **~5 h** |

After homework, take the [quiz](./quiz.md) and ship the [mini-project](./mini-project/README.md).
