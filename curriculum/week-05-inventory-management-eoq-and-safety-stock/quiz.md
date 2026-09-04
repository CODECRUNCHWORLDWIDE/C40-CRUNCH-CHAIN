# Week 5 — Quiz

Fifteen questions. Lectures closed. Aim for 13/15 before starting Week 6. A mix of multiple-choice and short computation — the answer key explains the *why*, not just the letter.

---

**Q1.** Which of the three inventory cost buckets does the classic EOQ model assume away entirely (assumes it never happens)?

- A) Holding cost
- B) Ordering cost
- C) Shortage cost
- D) Purchase cost

<details>
<summary>Answer</summary>

**C** — shortage cost. The classic EOQ model assumes demand is always met (no stockouts), isolating the holding-vs-ordering trade-off. Safety stock (Lecture 2) is what handles shortage cost.

</details>

---

**Q2.** A SKU has `D` = 2,000 units/year, `S` = $80/order, and `H` = $5/unit/year. What is its EOQ?

- A) 80 units
- B) 253 units
- C) 400 units
- D) 800 units

<details>
<summary>Answer</summary>

**B** — `EOQ = sqrt(2*2000*80/5) = sqrt(320000/5) = sqrt(64000) ≈ 252.98`, which rounds to **253 units**.

</details>

---

**Q3.** At `Q = Q*` (the EOQ), what is true about annual holding cost and annual ordering cost?

- A) Holding cost is always double ordering cost.
- B) They are equal.
- C) Ordering cost is always zero.
- D) There is no fixed relationship between them.

<details>
<summary>Answer</summary>

**B** — at the optimum, annual holding cost exactly equals annual ordering cost. This is a direct consequence of the calculus used to derive EOQ and a fast way to sanity-check any EOQ calculation.

</details>

---

**Q4.** According to the total-cost sensitivity formula from Lecture 1, ordering **20% more** than the true EOQ increases total cost by approximately:

- A) 20%
- B) 10%
- C) 1.7%
- D) 0%

<details>
<summary>Answer</summary>

**C** — about 1.7%, from `TC(Q)/TC(Q*) = 0.5*(r + 1/r)` with `r` = 1.20, giving a ratio of about 1.017. EOQ's total-cost curve is famously flat near its minimum.

</details>

---

**Q5.** A company reports a 95% cycle-service level. A colleague says this means "95% of all units of customer demand ship immediately." Is the colleague correct?

- A) Yes, that's exactly what cycle-service level means.
- B) No — that describes fill rate; cycle-service level is the probability of *no stockout at all* during a replenishment cycle, regardless of the stockout's size.
- C) No — cycle-service level only applies to newsvendor problems.
- D) Yes, but only for SKUs with constant demand.

<details>
<summary>Answer</summary>

**B** — that describes fill rate, a magnitude-weighted measure. Cycle-service level is a yes/no event probability per replenishment cycle, blind to how big any given stockout was.

</details>

---

**Q6.** In the demand-during-lead-time standard deviation formula `σ_DLT = sqrt(L*σ_d² + d̄²*σ_L²)`, which term usually dominates in real operations, and why?

- A) The demand-variability term, because daily demand is always more volatile than lead time.
- B) The lead-time-variability term, because it scales with mean daily demand, and a late delivery exposes you to extra days of full-rate demand.
- C) Neither — they are always equal by construction.
- D) The demand term, because `L` is always larger than `σ_L`.

<details>
<summary>Answer</summary>

**B** — the lead-time-variability term, `d̄² * σ_L²`, scales with mean daily demand squared: a late delivery on a fast-moving SKU exposes you to many extra days of full-rate demand, which is usually a bigger hit than ordinary day-to-day demand noise.

</details>

---

**Q7.** Which formula correctly computes safety stock for a target cycle-service level?

- A) `SS = z / σ_DLT`
- B) `SS = σ_DLT / z`
- C) `SS = z * σ_DLT`
- D) `SS = z + σ_DLT`

<details>
<summary>Answer</summary>

**C** — `SS = z * σ_DLT`. The z-score scales the demand-during-lead-time standard deviation up to the desired confidence level.

</details>

---

**Q8.** What `z`-score corresponds to a 95% cycle-service level target (assuming normally distributed demand during lead time)?

- A) 1.00
- B) 1.28
- C) 1.645
- D) 2.33

<details>
<summary>Answer</summary>

**C** — 1.645. (1.28 is 90%, 2.33 is 99%.)

</details>

---

**Q9.** The reorder point formula `ROP = d̄*L + SS` — what does the `d̄*L` term represent?

- A) The safety margin against uncertainty.
- B) The expected (average) demand during the lead-time window.
- C) The total annual demand.
- D) The EOQ.

<details>
<summary>Answer</summary>

**B** — expected (average) demand during the time you're waiting for the replenishment order to arrive. Safety stock is added on top to cover the *uncertainty* around that expectation.

</details>

---

**Q10.** Comparing an (s,Q) continuous-review policy to an (R,S) periodic-review policy with the same underlying demand and lead time, which generally needs **more** safety stock, and why?

- A) (s,Q), because it reorders more often.
- B) (R,S), because its exposure window is `R + L` instead of just `L` — you might have just missed a review.
- C) They always need identical safety stock.
- D) (R,S) never needs any safety stock at all.

<details>
<summary>Answer</summary>

**B** — (R,S) needs more safety stock, because its risk-exposure window is the review period plus lead time (`R+L`), not just lead time (`L`) — inventory can drop dangerously low right after a review and sit unprotected until the next one.

</details>

---

**Q11.** A base-stock (S-1,S) policy is best suited to:

- A) High-volume, cheap, fast-moving SKUs ordered in large batches.
- B) Expensive, slow-moving SKUs where holding extra safety stock is costly and demand is infrequent enough to review after every unit sold.
- C) One-time seasonal products with no reorder opportunity.
- D) Products with zero demand variability.

<details>
<summary>Answer</summary>

**B** — expensive, slow-moving SKUs where per-unit safety stock is costly and demand is infrequent enough to make "reorder after every sale" operationally realistic.

</details>

---

**Q12.** In the newsvendor model, the critical ratio is defined as:

- A) `CR = Co / Cu`
- B) `CR = Cu / Co`
- C) `CR = Cu / (Cu + Co)`
- D) `CR = (Cu + Co) / Cu`

<details>
<summary>Answer</summary>

**C** — `CR = Cu / (Cu + Co)`. This ratio is the quantile of the demand distribution that the optimal order quantity should target.

</details>

---

**Q13.** A newsvendor item has an underage cost `Cu` that is much larger than its overage cost `Co` (running out is far more costly than being stuck with extras). Relative to the mean demand forecast, the optimal order quantity `Q*` should be:

- A) Well below the mean.
- B) Exactly equal to the mean.
- C) Above the mean.
- D) Undefined — the newsvendor model doesn't apply when costs are unequal.

<details>
<summary>Answer</summary>

**C** — above the mean. When running out (`Cu`) is far costlier than overstocking (`Co`), the critical ratio is close to 1, giving a large positive `z` and pushing `Q*` well above the mean forecast.

</details>

---

**Q14.** In a multi-echelon network, when you aggregate demand across `n` independent locations at a central DC, which part of `σ_DLT` pools favorably (shrinks relative to the simple sum), and which does not?

- A) Both terms pool equally well.
- B) The demand-variability term pools favorably (variances add, so combined std dev grows slower than a simple sum); the lead-time-variability term does **not**, because it depends on *total* mean demand squared, and means add directly (no pooling benefit).
- C) Neither term ever pools — centralizing safety stock never helps.
- D) The lead-time-variability term pools favorably; the demand-variability term does not.

<details>
<summary>Answer</summary>

**B** — the demand-variability term pools favorably because independent variances add under a square root that grows slower than a linear sum; the lead-time-variability term does not pool, because it scales with the *square of total mean demand*, and means combine by simple addition, not by any variance-reduction mechanism.

</details>

---

**Q15.** A SKU has `annual_demand` = 3,600, mean daily demand `d̄` = 3,600/365 ≈ 9.86, `σ_d` = 4.0, `L` = 8 days, `σ_L` = 1.0 day. Compute `σ_DLT` (round to the nearest whole unit).

- A) About 6
- B) About 11
- C) About 15
- D) About 32

<details>
<summary>Answer</summary>

**C** — `σ_DLT = sqrt(L*σ_d² + d̄²*σ_L²) = sqrt(8*4² + 9.863²*1²) = sqrt(8*16 + 97.28) = sqrt(128 + 97.28) = sqrt(225.28) ≈ 15.0`, matching **About 15**.

</details>

**Scoring:** 13+ → start Week 6. 10–12 → re-read the lecture sections behind your misses. <10 → re-read all three lectures from the top; EOQ, safety stock, and reorder policy compound directly into everything from Week 6 onward.

---
