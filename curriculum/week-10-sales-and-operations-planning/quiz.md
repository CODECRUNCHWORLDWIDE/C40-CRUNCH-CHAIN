# Week 10 — Quiz

Fifteen questions. Lectures closed. Aim for 12/15 before moving to Week 11. A mix of multiple-choice and short "what would you compute" questions — the answer key at the bottom explains the *why*, not just the letter.

---

**Q1.** Put the four stages of the monthly S&OP cycle in the correct order:

- A) Supply review → Demand review → Executive sign-off → Reconciliation
- B) Demand review → Supply review → Reconciliation → Executive sign-off
- C) Reconciliation → Demand review → Supply review → Executive sign-off
- D) Executive sign-off → Demand review → Reconciliation → Supply review

---

**Q2.** Trail Footwear's April `stat_forecast_units` is 12,200 and `sales_input_units` is 13,400; the `consensus_forecast_units` on record is 12,800. The lecture's point about that number is that:

- A) It was computed automatically as `(stat + sales) / 2`, no judgment involved
- B) It's a documented, negotiated position tied to a stated assumption (a launch ramping to 50% run rate), not a formula result
- C) It's simply wrong and should be corrected to match `sales_input_units`
- D) `consensus_forecast_units` is always required to equal the average of the other two columns

---

**Q3.** The demand review's output and the supply review's output are, respectively:

- A) A capacity plan; a consensus forecast
- B) A consensus forecast; a statement of whether that forecast can be met and by how much
- C) A revenue target; a hiring plan
- D) They produce the same output, just from different departments

---

**Q4.** In a SQL running-balance query, why is `PARTITION BY product_family` required in the window function?

- A) It sorts the output alphabetically
- B) Without it, the running total would sum across all three families instead of resetting for each one
- C) It's optional — `ORDER BY` alone produces the same result
- D) It converts the query from a window function to a subquery

---

**Q5.** In the chase strategy, the dominant cost driver — the one that isn't present at all in a pure level strategy — is:

- A) Regular production cost
- B) Hiring and layoff cost, incurred repeatedly as capacity chases demand up and down
- C) Holding cost
- D) Overtime cost

---

**Q6.** In Lecture 2's worked example, chase and level strategies had **identical** total production cost ($195,000). Why?

- A) It's a coincidence specific to that dataset
- B) Both strategies produce the same total number of units over the horizon (equal to total demand), at the same regular unit cost — only the *timing* of production differs
- C) Level always costs more to produce because it uses overtime
- D) Chase never actually produces the full demand total

---

**Q7.** In that same worked example, level beat chase on total cost. The reason was:

- A) Level always beats chase in every possible cost structure
- B) In this specific cost structure, hiring/layoff cost was expensive relative to holding cost, so avoiding workforce churn was worth the extra inventory carried
- C) Chase is illegal under labor law
- D) Level uses less total production, which is always cheaper

---

**Q8.** The mixed strategy in Lecture 2's example ran a temporary negative ending inventory (a backorder) in month 4. This happened because:

- A) The simulation had a bug
- B) Even after ramping capacity and using maximum available overtime that month, the combination still fell short of demand, and the inventory buffer built up in earlier months wasn't large enough to cover the rest
- C) Mixed strategies can never avoid stockouts
- D) Month 4's demand was miscalculated

---

**Q9.** What does Integrated Business Planning (IBP) add on top of the four-stage S&OP cycle?

- A) A fifth approval stage
- B) Pulling Finance in as a full peer in the reconciliation — checking the plan against revenue and margin targets, not just units — and making the cycle more continuous
- C) IBP replaces the need for a demand review entirely
- D) IBP is just a rebranding with no functional difference from S&OP

---

**Q10.** A family can hit 100% of its unit revenue target and still have a real problem once you check margin. Why?

- A) Revenue and margin are always identical
- B) Hitting the volume target by leaning on overtime/subcontract units (which cost more per unit) can erode margin even while revenue looks on-target
- C) Margin can only be computed once a year
- D) This can never actually happen

---

**Q11.** Trail Footwear's April `consensus_forecast_units` is 12,800; April's maximum available supply (regular + overtime + subcontract) is 11,250. What does this tell you, on its own, before considering any earlier months?

- A) April is fully covered with room to spare
- B) Even using every unit of every lever available specifically in April, April alone cannot produce enough to meet its own forecast — a structural shortfall of 1,550 units
- C) The forecast must be wrong
- D) Overtime cost is too high to ever use

---

**Q12.** In Exercise 3, closing March's small gap first (bringing its ending inventory to exactly 900) still wasn't enough to prevent an April shortfall. Why not?

- A) March's fix should have made April's problem disappear completely, and if it didn't, the calculation was wrong
- B) March's fix only brought the balance up to the safety-stock floor, not a surplus — it left no *extra* buffer to help absorb April's much larger structural shortfall
- C) April's shortfall has nothing to do with any earlier month
- D) Overtime capacity resets to zero every month regardless of prior usage

---

**Q13.** Which of these is a well-defined scenario-planning assumption, in the sense Lecture 3 requires?

- A) "Downside: things go badly for Trail Footwear"
- B) "Downside: the April retail launch slips to Q3, so April-June demand reverts to the statistical baseline instead of the consensus forecast"
- C) "Downside: sales will probably be lower"
- D) "Downside: we should plan conservatively"

---

**Q14.** Put the four required parts of a gap-closing recommendation (Lecture 3, Section 6) in order:

- A) Recommendation → Cause → Number → Options
- B) Number → Cause → Costed options → Recommendation
- C) Options → Number → Recommendation → Cause
- D) Cause → Recommendation → Number → Options

---

**Q15.** The phrase "one set of numbers" refers to:

- A) A rule that every department must report identical dollar figures at all times, even before reconciliation
- B) The state, achieved after sign-off, where Sales, Ops, and Finance are all working from the same consensus forecast, capacity plan, and revenue target instead of three separately-adjusted versions
- C) A requirement that only one person may enter numbers into the S&OP system
- D) The single query used to generate the balance table

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **B** — Demand review, then supply review, then reconciliation, then executive sign-off, in that order; each stage's output feeds the next.
2. **B** — the consensus number is a documented negotiated position tied to a specific stated assumption, not a blind average or an automatic pick of one side.
3. **B** — demand review produces the one consensus forecast; supply review checks that forecast against real capacity and reports the size of any gap.
4. **B** — without `PARTITION BY product_family`, `SUM() OVER (ORDER BY month)` would run one continuous total across all rows regardless of family, corrupting every family's balance after the first.
5. **B** — chase's defining cost is repeated hiring/layoff as workforce size chases demand every month; level, by design, avoids this entirely by holding capacity constant.
6. **B** — both strategies, over the full horizon, produce a total number of units equal to total demand (chase by construction; level because production settled back to the starting inventory level by the end) — same units, same regular rate, same total production cost; only *when* those units get made differs.
7. **B** — the specific cost structure in the example (hiring $60/unit, holding only $4/unit/month) made avoiding workforce churn worth more than the extra inventory holding cost; a different cost structure could flip this result.
8. **B** — the ramp-plus-overtime combination in month 4 still fell short of demand, and the inventory buffer from months 1-2 (built more slowly under a moderate ramp than under level) wasn't big enough to fully absorb the rest.
9. **B** — IBP's addition is folding Finance in as a peer reconciling revenue/margin, not just units, and making the process more continuous rather than a single monthly event.
10. **B** — overtime and subcontract units cost more per unit than regular-time units, so hitting a volume/revenue target using expensive levers can still miss the margin target even while revenue looks fine.
11. **B** — this is a structural, single-month capacity ceiling problem: April cannot supply its own forecast even maxed out, independent of anything that happened in earlier months.
12. **B** — bringing March exactly to the 900-unit floor left zero surplus above that floor to carry forward; April's shortfall (thousands of units) is far larger than what one month's minimal fix could ever supply.
13. **B** — it names a specific, testable assumption change (the launch date slipping, and exactly what demand reverts to) rather than a vague sentiment.
14. **B** — Number, then Cause, then costed Options, then a stated Recommendation — in that order, every time.
15. **B** — "one set of numbers" describes the post-sign-off state where every function is working from the identical agreed plan, not a rule about who may type numbers into a system.

</details>

**Scoring:** 12+ → start Week 11. 9–11 → re-read the lecture sections behind your misses. <9 → re-read all three lectures from the top; reconciling demand, supply, and finance onto one plan is the foundation Week 11's risk-and-resilience work stress-tests directly.
