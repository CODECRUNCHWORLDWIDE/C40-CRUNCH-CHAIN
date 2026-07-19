# Week 11 — Homework

Five problems, ~5 hours total, spread across the week. These reinforce the lectures with a mix of hands-on SQL, a Python pipeline extension, and two written-reasoning tasks. Commit each.

All SQL runs against `network_risk_register` and `daily_ops` from the [README](./README.md) unless a problem says otherwise.

---

## Problem 1 — Twenty warm-up queries (75 min)

Write and run each. Put them in `warmups.sql` with a `-- N` comment and the answer beneath each.

**Against `network_risk_register`:**

1. Every risk, sorted by `impact` descending, then `likelihood` descending.
2. Count of risks per `mitigation_status`. *(Expected: `Partial` is the most common, at 9 of 16.)*
3. Every risk with `risk_category = 'Transportation'`.
4. The single risk with the lowest `risk_score` (`likelihood * impact`).
5. Every SPOF (`single_point_of_failure = TRUE`) with `annual_volume_share_pct >= 50`.
6. Average `impact` across all 16 rows, rounded to 2 decimals.
7. Every risk where `node_type = 'Facility'`.
8. The `risk_category` with the fewest rows.
9. Every risk with `likelihood = 2 AND impact = 3` (there's more than one — how many, and what does that tell you about score alone not being a unique identifier?).
10. All risks sorted by `annual_volume_share_pct` descending, `NULL`s last.

**Against `daily_ops`:**

11. Total row count per `dc`. *(Expected: 60 each.)*
12. The single highest `avg_lead_time_days` value in the whole table, and which `dc`/`ops_date` it belongs to.
13. Count of rows where `inbound_delay_flag = TRUE`, by `dc`.
14. Average `otif_pct` for each `dc`, across the full 60 days (disruption included).
15. The 5 dates with the lowest `fill_rate_pct` at Austin East.
16. Total `units_shipped` across the whole table, all DCs, all 60 days.
17. Every row where `otif_pct < 70` (severe-breach days).
18. The date range (`MIN(ops_date)`, `MAX(ops_date)`) covered by the table.
19. Average `orders_received` per `dc`, sorted descending.
20. Count of distinct `ops_date` values (should be 60, confirming one row per DC per day, no gaps).

---

## Problem 2 — Extend the risk register (60 min)

Make the register your own.

1. Write `INSERT` statements adding **3 new risks** to `network_risk_register` — invent plausible Crunch Gear risks not already covered (a fourth DC, a new carrier concentration, a specific cyber threat, a currency-exposure risk on a different sourcing country, etc.). Assign each a defensible `likelihood`, `impact`, `single_point_of_failure`, and `mitigation_status`.
2. Re-run Exercise 1's Task 1 (full ranked list) and Task 6 (must-mitigate-first shortlist) on the 19-row register. Did any of your new risks land in the top 5? In the "Mitigate now" quadrant?
3. **Deliver** `extend_register.sql` with the inserts and both re-run queries, plus 3-4 sentences justifying the likelihood/impact scores you assigned to your 3 new risks — a risk score is only useful if you can defend the number, not just assert it.

---

## Problem 3 — Explain the visibility lever (30 min)

In `visibility-writeup.md`, answer in prose (no more than 350 words total):

1. In your own words, why does a monthly reporting cycle understate a disruption that both begins and (partially) resolves within the same reporting month? Use the Lecture 2 Austin East April-average example (93.2%) in your explanation.
2. Exercise 3's pipeline detected the Andes Stitch Works disruption with a confirmed alert on 2026-04-29 — 5-6 days before the trough. If Crunch Gear's actual response time (from alert to action) is 2 days, how many days of "still getting worse" does the network avoid compared to detecting it only once the trough was already obvious?
3. Name one cost in Exercise 2's Task 7 total ($42/order-line, or the $20,250 expedite premium) that you'd expect to shrink if detection happened even earlier than 2026-04-29 — and explain the mechanism (why would earlier detection reduce that specific cost)?

---

## Problem 4 — A single-query DC risk scorecard (45 min)

In `dc_scorecard.sql`, write **one query** (one `SELECT`, CTEs allowed) against `daily_ops` that returns, for each `dc`: `dc`, `avg_otif` (across all 60 days), `min_otif`, `days_below_90_otif` (count of days with `otif_pct < 90`), and a `rank` column (`RANK()` or `ROW_NUMBER()`) ordering DCs worst-to-best by `days_below_90_otif`.

**Deliver** the query plus 2-3 sentences: does the DC with the lowest *average* OTIF across all 60 days necessarily rank worst on `days_below_90_otif`? (It might not — a DC with mediocre-but-consistent performance and a DC with excellent-then-catastrophic performance can land at similar averages while telling very different risk stories. Which pattern does Austin East show, and why does that distinction matter for a resilience review?)

---

## Problem 5 — A control-tower "what if" (60 min)

Crunch Gear's ops director asks: **"What if we'd had this pipeline running a year before the Andes Stitch Works fire — would daily monitoring alone have prevented it, or just caught it faster?"**

1. In `whatif.md`, answer directly: would a rolling z-score pipeline like Exercise 3's have *prevented* the fire itself? Why or why not? (This is really asking you to distinguish what visibility *can* and *can't* do — re-read Lecture 1, Section 6 on the four resilience levers if you're unsure.)
2. Using Challenge 1's expected-annual-loss framework, if Crunch Gear's actual response time to a confirmed alert is 2 days (drafting and approving an expedited air-freight order) instead of the ~12 days it effectively took this time (fire on 2026-04-15, first emergency air shipment presumably following shortly after the trough became undeniable around early May), estimate — with stated assumptions — how much of Exercise 2's $49,482 disruption cost a 2-day response time would have avoided. You don't need to be precise to the dollar; show your reasoning and a defensible estimate range.
3. **Deliver** `whatif.md` with both answers, 200-300 words total.

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
