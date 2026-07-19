# Exercise 2 — Simulate a Disruption

**Goal:** Measure the Andes Stitch Works fire — the disruption baked into `daily_ops` — with real numbers: how bad it got, how long it lasted, and what it cost, instead of "it was a rough month."

**Estimated time:** 90 minutes.

## Setup

Confirm the seed is loaded:

```sql
SELECT COUNT(*) FROM daily_ops;   -- must print 180
```

Recall the scenario from the [week README](../README.md): Andes Stitch Works (sole CMT line for Outerwear, [Week 6](../../week-06-procurement-and-supplier-analytics/)) has a fire on **2026-04-15**, down 18 days. Austin East's safety-stock buffer absorbs the first ~12 days; visible service damage runs **2026-04-27 through 2026-05-20**.

Create `solutions.sql` and put each answer under a `-- Task N` comment.

## Tasks

1. **Baseline.** Compute Austin East's average `otif_pct` and `fill_rate_pct` for all days **before** 2026-04-27 (the pre-disruption baseline). *(Expected: baseline OTIF ≈ 95.8%, baseline fill rate ≈ 96.9%.)*

2. **The trough.** Compute the same two averages, plus average `avg_lead_time_days`, restricted to **2026-05-04 through 2026-05-10** (the worst week). *(Expected: trough OTIF ≈ 62.1%, trough fill rate ≈ 67.0%, trough lead time ≈ 6.4 days.)*

3. **The single worst day, two ways.** Find the date with the single lowest `otif_pct` at Austin East, and (separately) the date with the single lowest `fill_rate_pct`. They are not the same day — report both dates and both values.

4. **Days in breach.** Crunch Gear's internal service-level floor is **90% OTIF**. Count how many days Austin East fell below that floor, and report the first and last date in that range. *(Expected: 20 days, 2026-04-29 through 2026-05-18.)*

5. **Recovery date.** Using a window function (`LAG()` twice, or `LEAD()`, to look at the current row plus the next two), find the **first date on or after 2026-05-10** where `otif_pct >= 95` for three consecutive days. That date is the disruption's official "recovered" mark. *(Expected: 2026-05-20.)*

6. **Excess unfulfilled order-lines.** This is the core cost-driver calculation, done in two steps:
   - **Step A:** Compute the *baseline* daily average of unfulfilled order-lines at Austin East — `AVG(orders_received * (1 - otif_pct/100.0))` — using only pre-disruption days (before 2026-04-27).
   - **Step B:** Compute the *actual* total unfulfilled order-lines — `SUM(orders_received * (1 - otif_pct/100.0))` — over the full disruption window, **2026-04-27 through 2026-05-20** (24 days).
   - **Step C (arithmetic, not SQL):** Multiply Step A's baseline rate by 24 days to get the "would have happened anyway" baseline total, then subtract that from Step B. This is the **excess** unfulfilled order-lines directly attributable to the disruption. *(Expected: Step A ≈ 6.6/day, Step B ≈ 855.4, baseline-for-24-days ≈ 159.0, excess ≈ 696.)*

7. **Dollar cost of the disruption.** Using Task 6's excess order-line count and these two given assumptions, compute the total disruption cost:
   - Each excess unfulfilled order-line costs Crunch Gear an average of **$42** in lost margin, rebooking discount, or absorbed customer-service cost.
   - To bridge the worst of the gap, Crunch Gear air-freighted **3 emergency shipments** of finished Outerwear from its backup CMT supplier (Pacific Rim Garments, [Week 6](../../week-06-procurement-and-supplier-analytics/)) via SkyBridge Air Cargo — **1,800 lb each**, at an incremental premium of **$3.75/lb** over the standard Ocean rate (per [Week 7](../../week-07-logistics-and-transportation-analytics/)'s Hai Phong lane figures: Air ≈ $3.85/lb, Ocean ≈ $0.10/lb).

   Compute: `(excess_order_lines × $42) + (3 × 1800 lb × $3.75/lb)`. *(Expected: ≈ $29,232 lost-margin cost + $20,250 expedite premium ≈ **$49,482 total**.)*

8. **Confirm the isolation.** Run Task 1 and Task 2's baseline-vs-trough comparison for `Memphis DC` and `Reno DC` instead of Austin East. Confirm both DCs show **no material change** between the two periods — proof the disruption stayed contained to the one DC dependent on the failed supplier.

## Expected result (spot checks)

- Task 1 → baseline OTIF ≈ 95.8%.
- Task 2 → trough OTIF ≈ 62.1%, trough lead time ≈ 6.4 days.
- Task 4 → 20 days in breach, 2026-04-29 to 2026-05-18.
- Task 5 → recovery date 2026-05-20.
- Task 6 → ≈696 excess unfulfilled order-lines.
- Task 7 → ≈$49,482 total disruption cost.

## Done when…

- [ ] `solutions.sql` has all 8 tasks under `-- Task N` comments.
- [ ] Your Task 7 total is within a few dollars of the spot check (small rounding differences in Task 6 are fine).
- [ ] You can state, in one sentence each: how bad it got (Task 2/3), how long it lasted (Task 4/5), and what it cost (Task 7).
- [ ] Task 8 confirms Memphis DC and Reno DC were essentially untouched.

## Stretch

- Recompute Task 7 under a "what if we'd caught it 7 days earlier" scenario: assume the excess order-line count would have been roughly 30% lower had Austin East escalated to emergency air freight a week into the visible impact window instead of during the trough. What's the revised total cost, and what does that number tell you about the dollar value of the visibility lever from Lecture 1?
- The 18-day production stoppage at Andes Stitch Works and the 24-day *visible* Austin East impact window don't match — visible impact started 12 days after the fire and lasted well past the supplier's 2026-05-03 restart. In your own words (3-4 sentences), explain the mechanism behind both gaps: why does a supplier outage take time to become visible downstream, and why does the visible impact outlast the supplier's actual recovery?

## Submission

Commit `solutions.sql` to your portfolio under `c40-week-11/exercise-02/`.
