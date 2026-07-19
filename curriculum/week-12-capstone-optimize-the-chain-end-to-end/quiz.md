# Week 12 — Quiz

Fifteen questions. Lectures closed. Aim for 13/15 before shipping the mini-project — this is the last quiz of the course. A mix of multiple-choice and short "what does this compute?" questions grounded in this week's actual capstone numbers; the answer key explains the *why*, not just the letter.

---

**Q1.** In Lecture 1's scoping document, why does the capstone explicitly hold plant-to-DC sourcing **out of scope**?

- A) Because plant sourcing can't be modeled in SQL
- B) Because it's a separate sourcing-strategy question with contract/tariff implications beyond a flow-optimization exercise, and stating that boundary up front is what a defensible scoping document does
- C) Because the LP solver can't handle more than two echelons
- D) Because plant costs are irrelevant to total landed cost

---

**Q2.** The capstone's baseline safety-stock policy is "a flat 30 days of extra supply, everywhere." What is the main problem with a flat day-count buffer rule, as this week's numbers demonstrate?

- A) It's illegal under GAAP
- B) It ignores each SKU-region's actual demand volatility and lead time, so it's almost always drastically oversized wherever lead times are short
- C) It's mathematically identical to `z·σ_LT` in every case
- D) It only works for seasonal products

---

**Q3.** In Lecture 1 §5's hand-worked baseline, why is Memphis DC's $38,000/month fixed cost being paid even though it ships zero units?

- A) It's a data-entry error in the seed table
- B) DC fixed costs (lease, staffing) are incurred regardless of volume — an idle, still-staffed DC is a real and common sign of an unoptimized network
- C) Memphis charges a penalty fee for being idle
- D) Fixed costs only apply to plants, not DCs

---

**Q4.** Why does the demand generator in Lecture 1 §4 use `np.random.seed(40)`?

- A) To make the demand numbers as large as possible
- B) To guarantee every SKU-region's demand is identical
- C) To make the "random" noise reproducible — anyone who runs the script gets the exact same 576 rows
- D) Seeding is required by `pandas.to_sql`

---

**Q5.** Lecture 2's backtested seasonal-naive forecast comes out to roughly 8–9% MAPE. Why is that specific number meaningful for this dataset?

- A) It's a coincidence with no significance
- B) 8% MAPE is the industry-standard benchmark for all forecasts
- C) It closely matches the 8% noise the demand generator injected on top of a fully-known seasonal pattern — confirmation the forecast is capturing the real seasonal signal and leaving only the irreducible noise as error
- D) It means the forecast is performing worse than a naive average

---

**Q6.** In the EOQ formula `EOQ = √(2DS/H)`, what does `H` represent?

- A) Total annual demand
- B) The fixed cost per replenishment order
- C) The annual holding cost per unit (unit cost × holding rate)
- D) The lead time in days

---

**Q7.** Lecture 2 §3 computes safety stock at MEM as `SS = Z · σ_d · √(LT_months)`. If MEM's lead time were **four times longer** (from 4 days to 16 days), and everything else stayed the same, safety stock would:

- A) Stay exactly the same
- B) Double
- C) Quadruple
- D) Increase by a factor of 16

---

**Q8.** What does `Z = 1.645` represent in the safety-stock formula?

- A) The average forecast error in units
- B) The standard normal value corresponding to a 95% cycle service level
- C) The number of standard deviations in a 100% guarantee against stockout
- D) A fixed constant that never changes regardless of target service level

---

**Q9.** In the network-flow LP (Lecture 2 §4), what does the constraint `sum(x[d,r] for d in DCS) == demand[r]` enforce?

- A) No DC may exceed its capacity
- B) Every region's demand must be fully met, summed across all DCs that ship to it
- C) Total cost must equal a fixed budget
- D) Every DC must ship an equal amount to every region

---

**Q10.** In the same LP, what does the constraint `sum(x[d,r] for r in REGIONS) <= capacity[d]` enforce?

- A) Every region gets exactly its forecasted demand from one DC only
- B) No single DC may ship more total units (summed across all regions) than its monthly capacity
- C) All DCs must ship the same total volume
- D) Freight cost must be minimized region by region, independent of capacity

---

**Q11.** The solved LP in Lecture 2 §4 routes 2,000 of Southeast's units through Austin East instead of Memphis DC, even though Memphis is the cheapest lane for Southeast. Why?

- A) Austin East has a lower fixed cost
- B) Memphis DC's 22,000-unit capacity is fully used by Northeast + Midwest + the rest of Southeast, and Southeast's marginal +$0.10/unit cost on Austin is the cheapest way to shed the 2,000 units Memphis can't hold
- C) The LP made an error; Memphis should have taken all of Southeast's demand
- D) Southeast's demand forecast was wrong

---

**Q12.** Across the capstone's two savings levers (network reallocation vs. safety-stock right-sizing), which contributes more to the $21,926/month total, per Lecture 2 §6?

- A) Network reallocation contributes all of it; safety stock contributes nothing
- B) They contribute exactly equal amounts
- C) Safety-stock right-sizing contributes the larger share (roughly 60%), driven mainly by how fast `√LT` shrinks as lead time shrinks
- D) Network reallocation contributes the larger share because it's the more complex calculation

---

**Q13.** Per Lecture 3, what is the single most common failure in a first analytics presentation to executives?

- A) Using too few charts
- B) Leading with the method (forecast details, LP formulation) instead of leading with the headline number and the ask
- C) Not including enough appendix material
- D) Presenting in Markdown instead of PowerPoint

---

**Q14.** In the worked memo (Lecture 3 §2), why does the "trade-off" section explicitly call out Memphis DC running at 100% capacity, rather than omitting it since the recommendation is still a clear net win?

- A) To make the memo look more thorough and longer
- B) Because every real recommendation costs something, and stating the trade-off unprompted — before a skeptical stakeholder asks — is what builds the credibility a hidden trade-off would cost you later
- C) Because 100% capacity utilization is always a bad outcome
- D) Because the memo template requires exactly one negative sentence

---

**Q15.** In Challenge 1 Part C, what does the "shadow price" of Memphis DC's capacity constraint represent?

- A) The list price Crunch Gear pays per square foot at Memphis DC
- B) The maximum amount it would be worth paying, per unit of extra capacity, to relax that binding constraint — read directly off the marginal cost increase when capacity is cut, or off the LP's constraint dual value
- C) A random number with no operational meaning
- D) The average freight cost across all lanes into Memphis

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **B** — the scoping document (Lecture 1 §2) holds plant sourcing fixed on purpose, because changing it opens contract and tariff questions outside this capstone's boundaried objective. Stating what's out of scope, and why, is exactly what a defensible scoping document does — it's not a limitation to hide.
2. **B** — a flat day-count rule never adjusts for actual demand volatility (`σ`) or actual lead time (`LT`); Lecture 2 §3 shows it comes out roughly 18× larger than the properly sized `z·σ_LT` policy once lead times are short, because it was never derived from either quantity in the first place.
3. **B** — fixed DC costs (lease, staffing, utilities) are paid whether or not a DC ships anything. An idle-but-staffed DC, still on the books at full fixed cost, is a textbook sign of an unoptimized network — exactly the kind of finding a capstone like this exists to surface.
4. **C** — `np.random.seed(40)` fixes the pseudo-random number generator's starting state, so every run of the script (yours, a classmate's, the grader's) produces byte-identical "random" noise and therefore identical demand numbers.
5. **C** — the generator injected exactly 8% noise on top of a known seasonal pattern (Lecture 1 §4); a correctly built seasonal-naive forecast recovers the seasonal signal and leaves only that irreducible noise as residual error, so its backtested MAPE landing near 8% is direct confirmation the forecast is working as well as it theoretically can on this dataset.
6. **C** — `H` is the annual holding cost per unit, typically `unit_cost × holding_rate`. `D` is annual demand, `S` is order cost, and lead time doesn't appear in the EOQ formula at all (it appears in the separate safety-stock formula).
7. **B** — safety stock scales with `√(LT)`, not `LT` directly. A 4× increase in lead time produces a `√4 = 2×` increase in safety stock, not a 4× or 16× increase — this is exactly why short lead times shrink safety stock so aggressively (Q2's answer, in reverse).
8. **B** — `Z = 1.645` is the standard normal distribution's value at the 95th percentile — the multiplier that, applied to lead-time demand's standard deviation, produces a safety-stock buffer sized to stock out no more than 5% of replenishment cycles. A 99% target uses a larger `Z` (≈2.326); it is not a fixed constant.
9. **B** — this is the demand-satisfaction constraint: summed across every DC that could ship to a region, total inbound units must exactly equal that region's forecasted demand. It's what guarantees the LP never "solves" a problem by quietly under-serving a region.
10. **B** — this is the capacity constraint: a single DC's total outbound volume, summed across all the regions it serves, cannot exceed its stated monthly throughput capacity.
11. **B** — Memphis's 22,000-unit capacity is exactly exhausted by Northeast (7,000) + Midwest (8,000) + 7,000 of Southeast's 9,000; the remaining 2,000 units of Southeast demand must go somewhere, and Austin's $1.30/unit Southeast rate (only $0.10 more than Memphis's $1.20) is the cheapest available overflow destination — a genuine LP-optimal answer, not an error.
12. **C** — per Lecture 2 §6's table, safety-stock right-sizing contributes $13,226 of the $21,926 total (≈60%), versus network reallocation's $8,700 (≈40%). The `√LT` relationship (Q7) means a flat day-count buffer is dramatically oversized once lead times are short, which is exactly this network's situation.
13. **B** — leading with the method instead of the headline number and the ask is the most common failure Lecture 3 names. Executives want the decision, not the derivation, in the first sentence.
14. **B** — omitting a real trade-off doesn't make the recommendation safer, it just moves the discovery of that trade-off to a moment when it looks like something you hid rather than something you accounted for. Stating it first is what earns trust for the *next* recommendation, too.
15. **B** — the shadow price (or dual value) of a binding constraint is the marginal value of relaxing it by one unit — here, how much cheaper the network would run for each additional unit of Memphis capacity, which directly answers "how much should we be willing to pay to keep that capacity."

</details>

**Scoring:** 13+ → ship the mini-project with confidence. 10–12 → re-read the lecture sections behind your misses before starting the mini-project. <10 → re-read all three lectures from the top; this week's numbers all connect to each other, and a shaky foundation here will surface as a shaky mini-project report.
