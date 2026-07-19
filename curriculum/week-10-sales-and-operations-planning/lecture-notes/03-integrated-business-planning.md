# Lecture 3 — Integrated Business Planning

> **Duration:** ~2 hours. **Outcome:** You can join the operational plan (units, capacity, inventory) to the financial plan (revenue, margin) in one SQL query, quantify exactly where the two disagree and why, build a simple upside/downside scenario alongside the base case, and write a gap-closing recommendation a finance leader would actually accept.

## 1. From S&OP to IBP

**Integrated Business Planning (IBP)** is what S&OP grows into once a company does two things: (1) makes the monthly cycle from Lecture 1 genuinely continuous — re-running key pieces of it whenever new information lands, not just once a month on a calendar — and (2) pulls **Finance fully into the process as a peer**, not a downstream recipient who gets handed a units plan and converts it to dollars after the fact. The mechanics you already know (consensus forecast, supply review, gap analysis) don't change. What changes is the scope of what gets reconciled: not just "can Ops make what Sales wants to sell," but **"does the resulting plan hit the revenue and margin numbers the business committed to."**

That reframing matters because units and dollars can disagree even when the units-level plan looks perfectly healthy. A family can hit 100% of its unit forecast and still miss its revenue target — if the mix shifted toward a lower-priced item, if a price increase didn't stick, or if the revenue target itself was built on a different (often more optimistic) volume assumption than the one Demand and Supply just agreed to. IBP is the discipline of catching that gap in the same monthly cycle, instead of discovering it in a quarterly earnings review three months later.

## 2. Two forecasts, two owners, one revenue number

Recall from the [week setup](../README.md): `financial_targets.revenue_target` was built by Finance during **last quarter's** budget cycle — before this month's consensus forecast existed. It is, in effect, **a third forecast**, built independently, on its own timeline, by a third owner. IBP's central move is comparing what that target *implies* about volume against what Demand and Supply have since agreed to:

```sql
SELECT f.month, f.product_family,
       f.revenue_target,
       ROUND(f.revenue_target / r.unit_price) AS implied_aop_units,
       d.consensus_forecast_units,
       ROUND(f.revenue_target / r.unit_price) - d.consensus_forecast_units AS unit_gap
FROM financial_targets f
JOIN product_reference r USING (product_family)
JOIN demand_plan d ON d.month = f.month AND d.product_family = f.product_family
WHERE f.product_family = 'Backpacks & Bags'
ORDER BY f.month;
```

```
 month      | revenue_target | implied_aop_units | consensus_forecast_units | unit_gap
------------+-----------------+--------------------+---------------------------+----------
 2025-01-01 |         460000  |               4842 |                      5100 |     -258
 2025-02-01 |         500000  |               5263 |                      5400 |     -137
 2025-03-01 |         620000  |               6526 |                      6800 |     -274
 2025-04-01 |         800000  |               8421 |                      8900 |     -479
 2025-05-01 |         970000  |              10211 |                     10200 |       11
 2025-06-01 |         910000  |               9579 |                      9600 |      -21
```

Backpacks & Bags' `implied_aop_units` runs consistently **below** the current consensus forecast for the first four months — Finance's target, built with older, more conservative assumptions, is now lower than what Demand and Sales have since agreed the family will actually sell. That's *good* news dressed up as a "gap": if Supply can cover the higher consensus number, the family is tracking to **beat** its January–April revenue targets, not miss them. Reporting only "we hit 100% of the AOP" would bury genuinely good news; reporting the gap explicitly is what lets Finance decide whether to raise the full-year target or hold it as a cushion against a worse month later.

## 3. When the operational plan can't deliver the revenue target

The more urgent version of this reconciliation is the reverse: what happens when a *supply constraint* — not a stale target — is what stands between the plan and the revenue number. Trail Footwear's April is exactly this case. Recall from Lecture 1's supply review: April's consensus forecast is 12,800 units, but April's **maximum** available capacity (regular + overtime + subcontract) is only 11,250 units — a structural shortfall of 1,550 units, *before* accounting for whatever buffer Q1 managed to pre-build.

```sql
SELECT d.month, d.consensus_forecast_units,
       s.regular_capacity_units + s.overtime_capacity_units + s.subcontract_capacity_units AS max_supply,
       r.unit_price,
       d.consensus_forecast_units * r.unit_price AS revenue_if_fully_supplied,
       LEAST(d.consensus_forecast_units,
             s.regular_capacity_units + s.overtime_capacity_units + s.subcontract_capacity_units)
             * r.unit_price AS revenue_at_max_capacity,
       f.revenue_target
FROM demand_plan d
JOIN supply_plan s USING (month, product_family)
JOIN product_reference r ON r.product_family = d.product_family
JOIN financial_targets f ON f.month = d.month AND f.product_family = d.product_family
WHERE d.product_family = 'Trail Footwear' AND d.month = '2025-04-01';
```

```
 month      | consensus_forecast_units | max_supply | unit_price | revenue_if_fully_supplied | revenue_at_max_capacity | revenue_target
------------+---------------------------+------------+------------+-----------------------------+---------------------------+-----------------
 2025-04-01 |                     12800 |      11250 |     145.00 |                  1856000.00 |                1631250.00 |         1700000
```

Even at **maximum** capacity utilization (regular + full overtime + full subcontract, all in April alone), Trail Footwear can only generate $1,631,250 in April revenue against a $1,700,000 target — a **$68,750 shortfall**, driven entirely by a physical capacity ceiling, not a demand problem. This is the number that has to go into the reconciliation meeting: not "April looks tight," but "April misses its revenue target by $68,750 *even in the best case*, unless capacity is pre-built ahead of the month." That specificity is what turns a vague warning into an actionable decision — and it's exactly why Lecture 1 flagged that Q1's surplus capacity needs to be used *deliberately*, not just enough to hit the safety-stock floor, to carry extra buffer into April. (You verified in Lecture 1 and will re-derive in [Exercise 3](../exercises/exercise-03-gap-analysis-and-actions.md) that using Q1's full overtime and subcontract capacity — not just regular capacity — closes this gap completely by the time April arrives.)

## 4. Margin, not just revenue

Revenue reconciliation alone can hide a real problem: hitting a revenue target with the *wrong* cost structure. Every unit closed with overtime or subcontract capacity costs more than a unit made on regular time — so a plan that hits its revenue number by leaning hard on overtime can still **miss its margin target**.

```sql
SELECT product_family, gross_margin_target_pct, unit_price, standard_unit_cost,
       ROUND(100.0 * (unit_price - standard_unit_cost) / unit_price, 1) AS standard_margin_pct
FROM product_reference;
```

```
 product_family    | gross_margin_target_pct | unit_price | standard_unit_cost | standard_margin_pct
--------------------+---------------------------+------------+---------------------+----------------------
 Trail Footwear     |                      55.0 |     145.00 |               62.00 |                 57.2
 Backpacks & Bags   |                      58.0 |      95.00 |               38.00 |                 60.0
 Apparel            |                      57.0 |     120.00 |               48.00 |                 60.0
```

Trail Footwear's **standard** margin (built on regular-time cost, $62/unit) is 57.2%, comfortably above its 55.0% target. But recall April's `unit_cost_overtime` is $31.00 and `unit_cost_subcontract` is $34.00 — barely above the $22.00 regular cost this week, so margin holds up fine even leaning on both. That's a deliberate teaching choice in this dataset; in many real businesses the overtime/subcontract premium is steep enough (2–3x regular cost) that hitting a revenue number the expensive way can blow straight through a margin target even while looking perfectly healthy on the top line. **Always check both** — a plan report that shows revenue-on-target and doesn't show the margin underneath it is an incomplete report.

### When margin actually breaks: a hypothetical contrast

This week's numbers are a deliberately gentle case — Trail Footwear's overtime premium ($31.00 vs. $22.00 regular, a 41% markup) barely dents a healthy 57.2% standard margin. Real overtime and subcontract premiums are often much steeper, and a *thinner*-margin family has far less room to absorb them. Compare two hypothetical product lines facing the identical 41% cost premium Trail Footwear actually has this week:

| | Trail Footwear (this week's real numbers) | A hypothetical thin-margin family |
|---|---|---|
| Unit price | $145.00 | $40.00 |
| Standard (regular-time) unit cost | $62.00 | $28.00 |
| Standard margin | 57.2% | 30.0% |
| Overtime unit cost (same 41% premium) | $31.00 vs. a $22.00 *regular production* cost | $39.48 vs. the $28.00 regular cost |
| Margin on an overtime-produced unit | `(145.00 - 31.00) / 145.00` ≈ **78.6%** | `(40.00 - 39.48) / 40.00` ≈ **1.3%** |

Same percentage premium, wildly different outcome: Trail Footwear's overtime units are still highly profitable, but the thin-margin family's overtime units are barely above break-even — one more cost bump (a second overtime shift, a rush freight surcharge to expedite the subcontracted units) and that family would be **losing money on every unit it makes to hit its own revenue target.** That's the exact trap Section 4's opening line warned about, made concrete: a revenue-target report that doesn't check margin underneath it would show this hypothetical family as "on plan" right up until the P&L reveals it barely made money doing it. **The tighter a family's standard margin, the more urgently its overtime/subcontract math needs checking — never assume this week's comfortable Trail Footwear numbers generalize to every family you'll ever reconcile.**

## 5. Scenario planning

A single-point forecast — one number per family per month — hides how much uncertainty is actually baked into a 6-month horizon. IBP teams routinely carry at least three parallel scenarios instead of one:

| Scenario | Definition | Typical use |
|---|---|---|
| **Base case** | The consensus forecast, as reconciled this week | The plan everyone executes against day to day |
| **Upside** | A defined "what if the good thing happens" — e.g., the April retail launch hits its *full* run rate instead of the assumed 50% | Tests whether capacity/supplier commitments could even support success |
| **Downside** | A defined "what if the bad thing happens" — e.g., the retail launch slips a full quarter | Tests whether the plan survives without a crisis; informs how much safety stock/flex capacity is worth paying for |

The discipline here is **defining each scenario as a specific, named assumption change**, not a vague "what if things go badly." "Downside: April retail launch slips to July" is testable and specific — you can literally rerun the balance-table query with `consensus_forecast_units` swapped for the stat-only number (12,200 instead of 12,800) and see exactly what changes. "Downside: things are worse" is not testable at all. [Challenge 2](../challenges/challenge-02-scenario-planning-model.md) has you build exactly this — three parallel forecast columns and a comparison of what each implies for capacity and revenue.

## 6. Common IBP failure modes

- **Revenue-only reconciliation.** Checking units and dollars but never rebuilding the margin math for a constrained month, exactly as Section 4 warned — a plan can look fully reconciled and still be quietly unprofitable.
- **Stale AOP treated as gospel.** Finance's `revenue_target` was right when it was built, months ago, on assumptions that have since changed (a new launch, a lost account). Treating it as immovable — instead of as one more input to reconcile, the way Section 2 treated it — turns every monthly cycle into an argument about whose number is "correct" instead of a shared update to the plan.
- **Scenario planning that never gets revisited.** A team builds a solid upside/downside model once, presents it, and then never checks back to see which scenario actually happened — losing the chance to calibrate how good their scenario assumptions actually were, and to reuse that calibration next quarter.
- **Confusing "reconciled" with "resolved."** IBP's job is to make every gap visible and costed — it does not make every gap disappear. A reconciled plan that still shows a $68,750 April shortfall is doing its job correctly; the failure mode is presenting that plan as if the gap isn't there.

## 7. Turning a gap into a recommendation

The output of all this reconciliation work is not a bigger spreadsheet of numbers — it's a **short, specific recommendation** that names the gap, its size, its cause, and the trade-off between the available fixes. A strong gap-closing writeup has four parts, every time:

1. **The number.** "Trail Footwear misses its April revenue target by $68,750 even at maximum April capacity."
2. **The cause.** "April's regular + overtime + subcontract ceiling (11,250 units) is below the 12,800-unit consensus forecast."
3. **The options, each costed.** "(a) Pre-build ~1,550 units of buffer using Q1's unused overtime/subcontract capacity, at an incremental cost of roughly $X; (b) accept the shortfall as a planned, communicated backorder; (c) push the April retail-launch ramp assumption back to Demand for revalidation."
4. **A recommendation, with the trade-off stated.** "Recommend (a): the incremental capacity cost is smaller than the lost-revenue risk of a stockout during a partner launch month, and it uses capacity that would otherwise sit idle in Q1."

That four-part shape — number, cause, costed options, recommendation — is what [Exercise 3](../exercises/exercise-03-gap-analysis-and-actions.md), both challenges, and the mini-project all grade you on. A gap report that stops at step 1 ("April looks bad") hasn't done the job; a recommendation without a stated cost for the alternative it rejected hasn't either.

## 8. Check yourself

- In your own words, what does IBP add to the four-stage S&OP cycle from Lecture 1?
- Why can a family "hit its revenue target" and still be a problem, once you look at margin?
- In the hypothetical thin-margin contrast (Section 4), why did the identical 41% overtime premium barely dent Trail Footwear's margin but nearly wipe out the other family's?
- What made Backpacks & Bags' January–April gap *good* news rather than bad news, and why would burying that gap by only reporting "% of target" be a mistake?
- Give one example each of a well-defined upside and downside scenario for Trail Footwear's April number, in the "specific, testable assumption" style this lecture requires.
- Name one IBP failure mode from Section 6 and explain what it looks like when a team falls into it.
- Name the four required parts of a gap-closing recommendation, in order.

If those are automatic, you're ready for the exercises — starting with building the running supply-demand balance table this lecture's queries have been previewing pieces of.

## Further reading

- **APICS/ASCM — Integrated Business Planning overview:** <https://www.ascm.org/>
- **Gartner — Sales and Operations Planning / IBP research summaries (general background, may require a free account):** <https://www.gartner.com/en/supply-chain>
- **PostgreSQL — Aggregate functions and `LEAST`/`GREATEST`:** <https://www.postgresql.org/docs/current/functions-conditional.html>
