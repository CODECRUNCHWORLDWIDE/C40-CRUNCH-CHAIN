# Exercise 2 — Compute Core KPIs by Hand

**Goal:** Compute OTIF, all three fill rate variants, perfect order rate, inventory turns/DIO, and cash-to-cash cycle time from raw data — by hand — until the formulas from Lecture 2 are muscle memory, not a lookup.

**Estimated time:** 1.5 hours.

## Setup

Below is one week of Crunch Gear DTC-channel order data — **not** the same table used in the lecture, so you can't just copy an answer. Create a file `kpis.md` and show your work for every task (numerator, denominator, and result) — a bare final number with no work shown doesn't demonstrate you understand the formula.

| order_id | promised_date | ship_date | qty_ordered | qty_shipped |
|---:|---|---|---:|---:|
| 1 | 2026-03-05 | 2026-03-05 | 120 | 120 |
| 2 | 2026-03-06 | 2026-03-08 | 80 | 80 |
| 3 | 2026-03-07 | 2026-03-07 | 150 | 140 |
| 4 | 2026-03-08 | 2026-03-08 | 60 | 60 |
| 5 | 2026-03-10 | 2026-03-12 | 100 | 90 |
| 6 | 2026-03-11 | 2026-03-11 | 200 | 200 |
| 7 | 2026-03-12 | 2026-03-12 | 75 | 75 |
| 8 | 2026-03-14 | 2026-03-15 | 90 | 90 |
| 9 | 2026-03-15 | 2026-03-15 | 130 | 130 |
| 10 | 2026-03-18 | 2026-03-18 | 110 | 100 |

And this quarter's finance snapshot (annualized figures, already computed for you — you only need to plug them in):

| Metric | Value |
|---|---:|
| Annual COGS | $1,800,000 |
| Average inventory value | $300,000 |
| Annual net sales | $2,500,000 |
| Average accounts receivable | $150,000 |
| Average accounts payable | $120,000 |

## Tasks

1. **On-time flag.** For each of the 10 orders, mark on-time (`ship_date <= promised_date`) or late. List which order IDs are late.

2. **In-full flag.** For each order, mark in-full (`qty_shipped >= qty_ordered`) or short. List which order IDs are short.

3. **OTIF.** Using tasks 1–2, compute OTIF% — orders that are *both* on-time and in-full, over total orders. Show the qualifying order IDs.

4. **Unit fill rate.** Sum `qty_ordered` and `qty_shipped` across all 10 orders, and compute unit fill rate.

5. **Order fill rate.** Compute the fraction of orders that shipped completely in full (ignore timing — use only task 2's flags).

6. **Compare fill rates.** In 2–3 sentences: why do unit fill rate and order fill rate come out to different numbers on the same data? Which one would a customer-experience-focused manager care about more, and why?

7. **Perfect order (simplified).** Assume none of these 10 orders had damage or invoicing errors (so perfect order here only depends on on-time + in-full, same as OTIF). State the perfect order rate under this assumption, and explain in one sentence what *additional* data you'd need to compute a *real* perfect order rate.

8. **Inventory turns.** Using the finance snapshot, compute annual inventory turns.

9. **Days of inventory (DIO).** Convert task 8's turns figure into days of inventory.

10. **DSO and DPO.** Using the finance snapshot, compute Days Sales Outstanding and Days Payable Outstanding.

11. **Cash-to-cash cycle time.** Combine tasks 9–10 into the full C2C formula. State the final number in days.

12. **Interpret it.** In 2–3 sentences: is a ~58-day cash-to-cash cycle good, bad, or "it depends"? What would you need to know to judge it (hint: compare to what, over what time)?

## Expected results (spot checks)

- Task 1 → late orders: **2, 5, 8**.
- Task 2 → short orders: **3, 5, 10**.
- Task 3 → OTIF = **50%** (5 of 10: orders 1, 4, 6, 7, 9).
- Task 4 → sum ordered = 1,115; sum shipped = 1,085; unit fill rate ≈ **97.3%**.
- Task 5 → order fill rate = **70%** (7 of 10).
- Task 8 → inventory turns = **6.0**/year.
- Task 9 → DIO ≈ **60.8 days**.
- Task 10 → DSO = **21.9 days**; DPO = **24.3 days**.
- Task 11 → C2C ≈ **58.4 days**.

## Done when…

- [ ] `kpis.md` shows numerator/denominator work for every task, not just final answers.
- [ ] Your OTIF, fill rate, and turns/DIO/C2C numbers match the spot checks above (small rounding differences are fine).
- [ ] Task 6 and Task 12 are answered in your own words, not copied from the lecture.

## Stretch

- Recompute OTIF using the *stricter* "on-time" definition of `ship_date < promised_date` (strictly before, not on-or-before). Does the number change? Should a real business use `<=` or `<` here — defend your choice in one sentence.
- If Crunch Gear cut its average inventory from $300,000 to $250,000 while holding COGS constant, what would the new inventory turns and DIO be? Would C2C improve, and by how many days?

## Submission

Commit `kpis.md` to your portfolio under `c40-week-01/exercise-02/`.
