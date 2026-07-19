# Week 8 — Quiz

Fifteen questions. Lectures closed. Aim for 13/15 before starting Week 9. A mix of multiple-choice and short computation — the answer key explains the *why*, not just the letter.

---

**Q1.** Put the six steps of the receive-to-ship flow in the correct order.

- A) Putaway → Receiving → Storage → Packing → Picking → Shipping
- B) Receiving → Putaway → Storage → Picking → Packing → Shipping
- C) Receiving → Storage → Putaway → Packing → Picking → Shipping
- D) Receiving → Picking → Putaway → Storage → Packing → Shipping

---

**Q2.** Picking typically accounts for roughly what share of total warehouse labor hours, and why does it dominate the other steps?

- A) ~10% — because it's the fastest step per unit of product handled.
- B) ~50% — because it happens once per order **line**, while receiving and shipping happen once per **shipment**, so its event count is far higher.
- C) ~50% — because pickers are paid a higher hourly wage than dock crews.
- D) ~90% — because receiving and shipping are almost fully automated in every warehouse.

---

**Q3.** Within a single pick, which component of pick time is typically the largest share, and which two levers taught this week directly reduce it?

- A) Search time; reduced by better slot labeling only.
- B) Pick-and-verify time; reduced by faster scanners only.
- C) Travel time (50–70% of pick time); reduced by slotting (Lecture 2) and batching (Lecture 3).
- D) Documentation time; reduced by removing paper pick lists.

---

**Q4.** What is the "golden zone" in a warehouse, and which SKUs should occupy it?

- A) The slots with the most cubic footage; assign the bulkiest SKUs there.
- B) The slots closest to the pack/ship station; assign the highest pick-frequency SKUs there.
- C) The slots farthest from the receiving dock; assign the most expensive SKUs there.
- D) A randomly rotating set of slots used to prevent picker boredom.

---

**Q5.** In standard ABC analysis, what cumulative-percentage cutoffs define Class A, B, and C?

- A) A = top 10 SKUs by count, B = next 10, C = the rest, regardless of percentage.
- B) A ≤ 80% cumulative share, B = 80–95%, C = the remaining 95–100%.
- C) A = exactly 20% of SKUs, B = exactly 30%, C = exactly 50%, always.
- D) A = SKUs picked every day, B = weekly, C = monthly or less, by calendar frequency.

---

**Q6.** A tiny 4-SKU catalog has pick-line counts of 30, 20, 10, and 5 (in that order, already sorted descending). Using the cumulative_pct ≤ 80 rule for Class A, which SKU is the **last** one still classified as Class A?

- A) The SKU with 30 picks (alone)
- B) The SKU with 20 picks
- C) The SKU with 10 picks
- D) The SKU with 5 picks

---

**Q7.** In the SQL window function `SUM(pick_line_count) OVER (ORDER BY pick_line_count DESC ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)`, what does this expression compute for each row?

- A) The grand total across all rows, repeated on every row.
- B) A running (cumulative) total from the top of the ranking down to and including the current row.
- C) The average of all rows seen so far.
- D) The difference between this row and the previous row.

---

**Q8.** When computing total pick-travel distance as `pick_line_count × 2 × distance_ft`, what does the `× 2` represent, and what happens if you omit it?

- A) It accounts for two pickers working simultaneously; omitting it double-counts labor.
- B) It converts feet to meters; omitting it gives the wrong unit.
- C) It converts a one-way slot distance into a round trip (there and back); omitting it understates true travel distance by half.
- D) It's a safety buffer for aisle congestion; omitting it has no effect on the total.

---

**Q9.** A SKU is picked 12 times this month from a slot 40 feet (one-way) from the pack station. What is its total monthly pick-travel distance?

- A) 480 feet
- B) 960 feet
- C) 40 feet
- D) 12 feet

---

**Q10.** To minimize total pick-travel distance across a catalog (given fixed pick frequencies and fixed slot distances), the optimal assignment pairs:

- A) SKUs sorted by pick frequency ascending with slots sorted by distance ascending.
- B) SKUs sorted by pick frequency descending with slots sorted by distance descending.
- C) SKUs sorted by pick frequency descending with slots sorted by distance ascending.
- D) SKUs in alphabetical order with slots in the order they were built.

---

**Q11.** What is the key operational difference between discrete (single-order) picking and batch (wave) picking?

- A) Discrete picking uses forklifts; batch picking never does.
- B) Discrete picking visits a slot once per order line that needs it; batch picking visits each distinct slot only once per wave, regardless of how many orders in the wave need it.
- C) Batch picking is always slower because it requires more totes.
- D) There is no operational difference — the terms are synonyms.

---

**Q12.** As pick-wave size increases (more orders batched together), what generally happens to per-line travel efficiency and to order cycle time?

- A) Both improve (get better) as wave size increases, with no trade-off.
- B) Travel efficiency tends to improve (more chances for orders to share a stop), but cycle time can suffer (orders wait longer for a bigger wave to fill before release) — a genuine trade-off.
- C) Travel efficiency gets worse as wave size increases; cycle time is unaffected by wave size.
- D) Wave size has no measurable effect on either metric.

---

**Q13.** A DC forecasts 950 order lines for a shift, at a target pick rate of 150 lines/hour. The shift is 8 hours with a 30-minute break (7.5 productive hours), and the target utilization is 80%. How many pickers are needed (rounding correctly)?

- A) 1
- B) 2
- C) 3
- D) 4

---

**Q14.** A picker completes all 5 lines of an order in 9 minutes flat. The order then sits in queue for pack for 4 minutes, is packed in 6 minutes, and then waits 45 minutes staged at the dock before its truck departs. What is this order's **order cycle time**, and why is "pick time" alone a misleading proxy for it?

- A) 9 minutes — order cycle time only counts active picking.
- B) 64 minutes — order cycle time is the full elapsed time from release to ship, and in this example the dock-wait time (45 of the 64 minutes) dwarfs the actual pick time, which is a common real-world finding.
- C) 15 minutes — pick time plus pack time only; dock wait doesn't count.
- D) 45 minutes — only the largest single component counts.

---

**Q15.** "Pick productivity" and "pick rate" (as used in this week's labor-sizing formula) refer to essentially the same underlying number, viewed from two directions. Which pair of descriptions correctly captures that relationship?

- A) Pick rate is measured in dollars; pick productivity is measured in feet — they are unrelated.
- B) Pick rate is a forward-looking planning input (output expected per labor-hour, used to size a crew); pick productivity is a backward-looking audit metric (output actually achieved per labor-hour, used to check the plan against reality) — same units, different direction of use.
- C) Pick rate only applies to batch picking; pick productivity only applies to discrete picking.
- D) They measure completely different things: pick rate is about SKUs, pick productivity is about orders.

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **B** — Receiving → Putaway → Storage → Picking → Packing → Shipping. Product enters once, is stored, then is picked, packed, and shipped out.
2. **B** — roughly half of warehouse labor hours, because picking scales with **order-line count**, which is almost always far higher than shipment count (Lecture 1's Austin East example: 225 pick lines from a handful of inbound receipts).
3. **C** — travel time, 50–70% of a pick, reduced by better slotting (put things closer, Lecture 2) and by batching (fewer trips for the same work, Lecture 3).
4. **B** — the slots closest to the pack/ship station; the highest-frequency SKUs belong there because every trip to that slot is multiplied by how often it's picked.
5. **B** — A ≤ 80% cumulative share, B = 80–95%, C = the remaining tail. The *SKU-count* split (like "20% of SKUs") is a common outcome, not the rule itself — the rule is the cumulative percentage.
6. **B** — the SKU with 20 picks. Cumulative: 30→(30/65=46.2%), 30+20=50→(76.9%), 30+20+10=60→(92.3%). The 20-pick SKU's cumulative (76.9%) is still ≤80%; the next SKU (10 picks) pushes it to 92.3%, over the cutoff, so it lands in Class B.
7. **B** — a running (cumulative) total: for each row, it sums every row from the top of the ranking down through the current row, which is exactly what a cumulative-share calculation needs.
8. **C** — it converts a one-way slot distance into a round trip. A pick means walking to the slot *and back*; omitting the `×2` understates every travel calculation in this week by exactly half.
9. **B** — `12 × 2 × 40 = 960` feet.
10. **C** — sort SKUs by pick frequency descending, sort slots by distance ascending, pair them position-for-position. This is the assignment-problem solution for minimizing total (frequency × distance) cost.
11. **B** — discrete picking makes one trip to a slot for every order line that needs it; batch picking consolidates all orders in a wave so each distinct slot is visited once per wave no matter how many orders in that wave need it.
12. **B** — bigger waves generally improve travel efficiency (more overlap, fewer trips) but can hurt cycle time, because an order has to wait for its wave to be released, and a bigger wave takes longer to fill. This is a real trade-off, not a free win in either direction.
13. **B** — 2 pickers. Pick rate 150 lines/hr = 0.4 min/line. Required minutes = 950 × 0.4 = 380. Usable minutes/picker at 80% of 450 productive minutes = 360. Pickers = ceil(380/360) = ceil(1.056) = **2** — even a small overage rounds up to a full extra picker, because a fraction of a picker isn't schedulable.
14. **B** — 64 minutes (9 + 4 + 6 + 45), the full release-to-ship elapsed time. Pick time alone (9 minutes) badly understates the real customer-facing cycle time — in this example, dock-wait time is by far the largest component, a very common real-world pattern.
15. **B** — pick rate is the forward-looking planning number (what you expect a picker to produce per hour, used to size a crew for a forecast); pick productivity is the backward-looking audit number (what a picker actually produced per hour, used to check the plan against reality). Same measurement, opposite direction of use.

</details>

**Scoring:** 13+ → start Week 9. 10–12 → re-read the lecture sections behind your misses. <10 → re-read all three lectures from the top; ABC classification, slotting, and labor sizing compound directly into everything from Week 9 onward.
