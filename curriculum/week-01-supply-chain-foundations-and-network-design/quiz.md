# Week 1 — Quiz

Fifteen questions. Lectures closed. Aim for 12/15 before starting Week 2. A mix of multiple-choice and short "compute this" questions — the answer key at the bottom explains the *why*, not just the letter.

---

**Q1.** In supply chain terms, an "echelon" is best described as:

- A) A single company's finance department
- B) A tier or stage the product physically passes through and stops at
- C) A synonym for "warehouse" only
- D) The final customer's location

---

**Q2.** Which of the three flows (product, information, cash) typically moves in **both** directions through a chain?

- A) Product flow only
- B) Cash flow only
- C) Information flow
- D) None of them — all three move only downstream

---

**Q3.** At the moment a container of finished jackets is mid-ocean, which is most likely already true?

- A) The retailer has already paid Crunch Gear in full.
- B) Crunch Gear has likely already paid the factory (deposit or letter of credit), before the retailer has paid anything.
- C) No cash has moved anywhere in the chain yet.
- D) The consumer has already bought and received the jacket.

---

**Q4.** The "decoupling point" in a supply chain is:

- A) Where the company's legal ownership of goods changes
- B) The point where the process switches from forecast-driven (push) to order-driven (pull)
- C) The exact geographic center of the distribution network
- D) The point where a shipment crosses an international border

---

**Q5.** Ten orders: 8 shipped on time, 2 shipped late. Of the 8 on-time orders, 1 was short-shipped. All other orders shipped in full. What is OTIF?

- A) 80%
- B) 70%
- C) 90%
- D) 100%

---

**Q6.** Using the same 10 orders as Q5 (assume total units ordered = 1,000 and total units shipped = 970), what is the **unit fill rate**?

- A) 70%
- B) 80%
- C) 97%
- D) 100%

---

**Q7.** Why can unit fill rate be much higher than order fill rate on the same dataset?

- A) They are always identical by definition.
- B) Unit fill rate only counts large orders; order fill rate only counts small ones.
- C) A few large shortages spread across many small orders can still leave the *total unit* shortfall small, while many *individual orders* still fail to ship complete.
- D) Order fill rate ignores quantity entirely.

---

**Q8.** If OTIF, damage-free rate, and invoice-accuracy rate are each independently 92%, the resulting perfect order rate is closest to:

- A) 92%
- B) 78%
- C) 96%
- D) 100%

---

**Q9.** Why does perfect order use multiplication of its components rather than an average?

- A) Multiplication is easier to compute than an average.
- B) An order only counts as "perfect" if it succeeds on every dimension simultaneously — one failure fails the whole order, which is what multiplying independent success probabilities models.
- C) Averaging is mathematically invalid for percentages.
- D) It doesn't — perfect order is always the average of its components.

---

**Q10.** Annual COGS = $4,000,000. Average inventory value = $500,000. What are the inventory turns?

- A) 12.5
- B) 8.0
- C) 0.125
- D) 4.0

---

**Q11.** Using Q10's answer, approximately what is the days of inventory (DIO)?

- A) ~29 days
- B) ~46 days
- C) ~91 days
- D) ~365 days

---

**Q12.** In the cash-to-cash cycle formula `C2C = DIO + DSO − DPO`, why is DPO **subtracted** while DIO and DSO are added?

- A) DPO is always a negative number.
- B) DIO and DSO represent days cash is tied up; DPO represents days the company still holds cash it owes but hasn't paid yet, which offsets the other two.
- C) It's an arbitrary convention with no underlying meaning.
- D) DPO should actually be added too — the formula as written is a simplification not used in practice.

---

**Q13.** According to the square root law of inventory pooling, if a company splits its safety stock from 1 central location into 9 regional locations (of similar size), total safety stock needed is expected to roughly:

- A) Stay exactly the same
- B) Double (×2)
- C) Triple (×3)
- D) Increase ninefold (×9)

---

**Q14.** A cross-dock facility differs from a standard distribution center primarily because:

- A) It's always larger than a standard DC.
- B) It receives inbound freight and reloads it onto outbound trucks the same day, with little or no inventory put away into storage.
- C) It only handles international shipments.
- D) It never handles perishable goods.

---

**Q15.** On the cost-service trade-off curve, moving from 99% to 99.9% service level is typically:

- A) Roughly the same cost as moving from 90% to 91%
- B) Cheaper than moving from 90% to 95%, because the network is already well-optimized
- C) Disproportionately more expensive than earlier gains, because it usually requires standing expedited freight and safety stock against rare, extreme demand spikes
- D) Free, because service level and cost are unrelated past 99%

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **B** — an echelon is a tier/stage the product physically passes through, not a department or a single warehouse specifically.
2. **C** — information flow moves both downstream (ASNs, shipment notices) and upstream (POS data, orders, forecasts). Product flows downstream; cash flows upstream.
3. **B** — payment to the factory (deposit or letter of credit) typically happens before or at production/shipment, well before the retailer has received and sold the goods and paid Crunch Gear.
4. **B** — the decoupling point is where forecast-driven push work meets order-driven pull work; it's usually wherever finished-goods inventory sits waiting for a real order.
5. **B** — OTIF requires both on-time AND in-full. Of the 10 orders, 2 shipped late (fail), and of the remaining 8 on-time orders, 1 was short-shipped (fail). That leaves 7 orders that are both on-time and in-full: **OTIF = 7/10 = 70%**. *(If you picked C — 90% — that's just the on-time rate; it ignores the short-shipped order.)*
6. **C** — 970/1,000 = 97%. Unit fill rate only cares about total units, not how they were distributed across orders.
7. **C** — a shortfall concentrated on a few orders barely dents the total unit count but can fail many individual orders' fill status — exactly why order-level and unit-level fill rate diverge.
8. **B** — 0.92 × 0.92 × 0.92 ≈ 0.779, or about **78%**. Three "pretty good" 92% rates compound down substantially once ANDed together.
9. **B** — perfect order is a strict AND across every dimension; one failure fails the whole order, which is exactly what multiplying independent success rates computes.
10. **B** — 4,000,000 / 500,000 = **8.0** turns/year.
11. **B** — 365 / 8.0 ≈ **45.6 days**, closest to "~46 days."
12. **B** — DIO and DSO are days cash is tied up (paid out, not yet recovered); DPO is days the company still holds cash it owes suppliers but hasn't paid — offsetting, not adding to, the cash tied up elsewhere.
13. **C** — √9 = 3, so total safety stock scales roughly ×3, not ×9 and not staying flat. This is the whole point of the square root law: splitting into more locations costs *less* extra safety stock than a naive "n locations = n times the stock" assumption, but still costs meaningfully more than staying centralized.
14. **B** — same-day reload with little/no storage is the defining feature of a cross-dock; it captures consolidation benefits without paying for inventory to sit.
15. **C** — the cost-service curve rises steeply at the high end; squeezing out the last fraction of a percent of service typically requires standing expedited freight and rich safety stock against rare extreme spikes, disproportionate to the service gain.

</details>

**Scoring:** 12+ → start Week 2. 9–11 → re-read the lecture sections behind your misses. <9 → re-read all three lectures from the top; these five KPIs and the network-design framework get reused in nearly every week that follows.
