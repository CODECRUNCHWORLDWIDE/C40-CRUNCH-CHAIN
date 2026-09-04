# Week 6 — Quiz

Fourteen questions. Lectures closed. Aim for 11/14 before starting Week 7. A mix of multiple-choice and short "compute this" questions — the answer key explains the *why*, not just the letter.

---

**Q1.** A "spend cube" is best described as:

- A) A single supplier's total annual spend
- B) Purchase data organized so it can be sliced along multiple dimensions at once — category, supplier, business unit, time
- C) A 3D chart type used only for spend data
- D) The total dollar amount a company spends in a fiscal year

<details>
<summary>Answer</summary>

**B** — a spend cube is spend data structured so it can be sliced along multiple dimensions (category, supplier, BU, time) in combination, not a single number or chart type.

</details>

---

**Q2.** Crunch Gear's top 4 suppliers (of 15) account for 81.5% of total spend. This pattern is most accurately described as:

- A) A data-entry error — spend should be evenly distributed across all suppliers
- B) Irrelevant to procurement strategy
- C) A classic Pareto (80/20) concentration — it signals both negotiating leverage and single-supplier risk, simultaneously
- D) Proof that Crunch Gear has too many suppliers and should immediately cut the other 11

<details>
<summary>Answer</summary>

**C** — 80/20 concentration is a double-edged signal: it means real negotiating leverage with the top suppliers, and real exposure if one of them has a bad quarter. It is not inherently an error or automatically a reason to cut suppliers.

</details>

---

**Q3.** "Tail spend" refers to:

- A) Spend that occurs only at the end of the fiscal year
- B) The long list of small, fragmented purchases spread across many vendors, individually too small to justify a negotiated contract
- C) Spend on shipping and freight specifically
- D) Any purchase order under $100

<details>
<summary>Answer</summary>

**B** — tail spend is the long list of small, fragmented, individually-too-small-to-negotiate purchases, not a time period or dollar threshold by itself.

</details>

---

**Q4.** How is "maverick buying" different from "tail spend" in general?

- A) They are exactly the same thing with different names
- B) Maverick buying is always illegal; tail spend is not
- C) Tail spend describes the size/fragmentation pattern of purchases; maverick buying specifically describes purchases made *off-contract*, bypassing negotiated agreements — tail spend is where maverick buying most often hides, but not all tail spend is maverick
- D) Maverick buying only applies to indirect/MRO categories, never direct materials

<details>
<summary>Answer</summary>

**C** — tail spend describes a *size/fragmentation pattern*; maverick buying describes purchases made *off-contract*. They overlap heavily in practice (most maverick spend is small and fragmented) but are conceptually distinct — a large purchase could technically be maverick too if it bypassed a negotiated agreement.

</details>

---

**Q5.** Three suppliers have defect rates of 1%, 3%, and 5%. Using min-max normalization where lower defect rate is better, what quality score (0–100) does the supplier with a 3% defect rate receive?

- A) 0
- B) 30
- C) 50
- D) 100

<details>
<summary>Answer</summary>

**C** — `100 × (worst − value) / (worst − best) = 100 × (5 − 3) / (5 − 1) = 100 × 2/4 = 50`.

</details>

---

**Q6.** A supplier scorecard's on-time delivery (OTD) rate and lead-time reliability (standard deviation of lead time) measure:

- A) The exact same thing, just expressed in different units
- B) Two genuinely different things — OTD is a per-order yes/no against a promised date; lead-time reliability measures how much actual lead time swings order to order, independent of what was promised
- C) OTD only matters for international suppliers; lead-time reliability only matters for domestic ones
- D) Lead-time reliability is a component of the cost score, not a separate metric

<details>
<summary>Answer</summary>

**B** — OTD is a per-order pass/fail against a promise date; lead-time reliability measures the swing in *actual* lead time regardless of what was promised. A supplier can hit 100% OTD by quoting a generous promise date while still being unpredictable to plan around.

</details>

---

**Q7.** A supplier scores a perfect 100.0 on a weighted scorecard built from only 2 purchase orders. The most appropriate response is to:

- A) Immediately award them all future volume in that category — a perfect score is a perfect score
- B) Ignore the scorecard entirely since it's clearly wrong
- C) Treat the score as a genuine but low-confidence signal — flag the small sample size, and gather more orders' worth of data before making a major volume-shifting decision
- D) Average their score with 50 to "correct" for the small sample

<details>
<summary>Answer</summary>

**C** — the score is real information, but a 2-order sample carries much less statistical confidence than a 10-order sample; the right move is to flag it and gather more data before a major decision, not to ignore it or blindly act on it.

</details>

---

**Q8.** Which of the following is **not** one of the four components in this week's TCO model?

- A) Unit price
- B) Freight cost per unit
- C) Supplier's headquarters location
- D) Carrying cost of extra safety stock driven by lead-time variability

<details>
<summary>Answer</summary>

**C** — headquarters location isn't one of the four TCO components in this week's model (unit price, freight/unit, quality cost/unit, carrying cost/unit). It might *influence* freight cost, but it isn't itself a TCO line item.

</details>

---

**Q9.** A part has a unit price of $10.00, freight cost of $0.50/unit, quality (rework) cost of $0.20/unit, and carrying cost of $0.10/unit. What is the total cost of ownership (TCO) per unit?

- A) $10.00
- B) $10.50
- C) $10.70
- D) $10.80

<details>
<summary>Answer</summary>

**D** — `10.00 + 0.50 + 0.20 + 0.10 = 10.80`.

</details>

---

**Q10.** Using the safety-stock formula `SS = z × daily_demand × σ_lead_time`, with z = 1.65, daily demand = 40 units, and a lead-time standard deviation of 3 days, what is the required safety stock?

- A) 66 units
- B) 120 units
- C) 198 units
- D) 480 units

<details>
<summary>Answer</summary>

**C** — `1.65 × 40 × 3 = 198` units.

</details>

---

**Q11.** Two suppliers have the *same* average lead time of 30 days. Supplier A has a lead-time standard deviation of 2 days; Supplier B has a standard deviation of 6 days. All else equal, which requires more safety stock, and why?

- A) Supplier A, because a shorter average always means more safety stock
- B) Supplier B, because higher lead-time variability — not average lead time — is what drives the safety-stock requirement
- C) Neither — safety stock only depends on average lead time, not variability
- D) They require identical safety stock since their averages match

<details>
<summary>Answer</summary>

**B** — safety stock (for lead-time variability) is driven by the *standard deviation* of lead time, not its average; Supplier B's higher σ (6 vs. 2 days) requires substantially more safety stock even though both suppliers average the same 30-day lead time.

</details>

---

**Q12.** Which statement best describes the trade-off between single-sourcing and dual-sourcing a category?

- A) Dual-sourcing is always better because it's never worth accepting any supplier concentration risk
- B) Single-sourcing is always better because splitting volume always increases total cost with no offsetting benefit
- C) Single-sourcing typically maximizes negotiating leverage and unit-price advantage but concentrates supply risk; dual-sourcing typically costs somewhat more per unit but buys supply continuity if one supplier fails
- D) The choice has no effect on price, only on delivery speed

<details>
<summary>Answer</summary>

**C** — single-sourcing typically wins on price/leverage but concentrates risk; dual-sourcing typically costs a bit more but buys continuity. Neither extreme (A or B) is correct as a blanket rule.

</details>

---

**Q13.** A should-cost model estimates a CMT supplier's fee should be about $17.28/unit; the supplier actually charges $22.46/unit — a 30% gap. The most accurate interpretation is:

- A) The supplier is committing fraud and should be reported
- B) The should-cost model is obviously broken and should be discarded
- C) This is a realistic, usable negotiation range — should-cost models are deliberately conservative and don't capture every real cost a factory carries, but a 15–30% gap is a legitimate starting point for a pricing conversation
- D) The gap proves Crunch Gear should switch suppliers immediately, without further analysis

<details>
<summary>Answer</summary>

**C** — should-cost models are deliberately conservative estimates, not audits; a 15–30% gap is a normal, usable negotiation range. Only a wildly large gap (200%+) would suggest the model's assumptions, not the supplier, are the problem.

</details>

---

**Q14.** A price-variance report shows a company's *net* variance across all suppliers is exactly $0 — actual spend matched standard cost overall. What is the most important follow-up question?

- A) None — a net variance of $0 means everything is fine and no further analysis is needed
- B) Whether that $0 net is hiding large offsetting favorable and unfavorable variances at individual suppliers, which would need very different follow-up actions
- C) Whether the company should stop tracking price variance altogether since it nets to zero
- D) Whether the accounting team made an error, since variance should never be exactly zero

<details>
<summary>Answer</summary>

**B** — a net variance near zero across multiple suppliers can easily mask a large unfavorable variance at one supplier fully offset by a large favorable variance at another — two very different follow-up conversations that a net-only view would hide entirely.

</details>

**Scoring:** 11+ → start Week 7. 8–10 → re-read the lecture sections behind your misses. <8 → re-read all three lectures from the top; the scorecard-normalization and TCO/safety-stock math get reused in nearly every remaining week of this course.

---
