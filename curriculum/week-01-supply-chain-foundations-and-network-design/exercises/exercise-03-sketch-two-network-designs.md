# Exercise 3 — Sketch Two Network Designs

**Goal:** Apply Lecture 3's centralize-vs-distribute framework to a concrete growth scenario — sketch both options, reason about the square root law and the cost-service curve, and produce a real (if rough) numeric comparison.

**Estimated time:** 1 hour.

## Setup

**The scenario:** Crunch Gear's DTC (direct-to-consumer) channel has grown fast. It currently ships every online order from a single DC in Memphis, Tennessee. Average delivery time nationwide is 4–6 days. Leadership is deciding whether to open regional forward-stocking locations to speed up delivery, and has asked you — the newest member of the ops analytics team — to sketch both options and give a recommendation.

Known facts to work with:

- Current single-DC safety stock (to protect service nationwide): **$500,000**.
- Annual carrying cost rate for inventory: **22%** of value (financing, insurance, obsolescence risk, storage — a standard industry rule-of-thumb range is 15–30%; we're using 22%).
- Fixed annual cost to run the Memphis DC: **$420,000**.
- Fixed annual cost to run **one** additional regional DC (lease, labor, systems), sized smaller than Memphis: **$180,000** each.
- Rough estimate: distributing to 4 regional DCs would let Crunch Gear cut average delivery time from 4–6 days to 1–2 days for most of the country.

Create a file `networks.md` for your answers.

## Tasks

1. **Sketch Design A — centralized.** Draw the single-DC network (Memphis → customers nationwide) in the ASCII style from Lecture 1/3. Label the one facility and note its approximate service reach (which regions get fast delivery, which get slow).

2. **Sketch Design B — distributed.** Draw a 4-regional-DC network (e.g., Northeast, Southeast, Midwest, West) feeding customers in their own zone, each fed in turn from the factory/inbound side. Label all 4 facilities.

3. **Apply the square root law.** Using Lecture 3 §3, estimate the new *total* safety stock required across the 4 regional DCs in Design B. Show the `√n` calculation.

4. **Compute carrying cost for both designs.** Using the 22% carrying-cost rate, compute the annual inventory carrying cost for Design A's safety stock and Design B's (from task 3).

5. **Compute total facility + carrying cost for both designs.** Add fixed facility cost + inventory carrying cost for each design. Which is cheaper on this cut alone?

6. **Name the missing piece.** Section 5's comparison leaves out outbound delivery cost and the value of faster delivery (potential lost/won sales from service level, per Lecture 3 §5–6). In 2–3 sentences, describe what data you'd need to fill that gap and finish a real total-landed-cost comparison.

7. **Make a recommendation — with a condition.** Write 3–5 sentences recommending Design A or B, but frame it as conditional: *"Choose B if [condition about delivery-speed sensitivity / competitive pressure / customer segment], otherwise choose A."* A confident "it depends, and here's specifically what it depends on" is a stronger answer than a flat pick.

## Expected results (spot checks)

- Task 3 → `√4 = 2`, so Design B's total safety stock ≈ **$1,000,000** (double Design A's $500,000).
- Task 4 → Design A carrying cost = $500,000 × 22% = **$110,000**; Design B carrying cost = $1,000,000 × 22% = **$220,000**.
- Task 5 → Design A total = $420,000 + $110,000 = **$530,000**; Design B total = (4 × $180,000) + $220,000 = $720,000 + $220,000 = **$940,000**. Design A is cheaper by **$410,000/year** on this cut alone — before counting the delivery-speed benefit of Design B.

## Done when…

- [ ] `networks.md` has both diagrams, clearly labeled.
- [ ] The square root law calculation is shown, not just stated.
- [ ] Task 5's totals match the spot checks (or you can explain a deliberate, stated deviation).
- [ ] Your recommendation in task 7 names a specific condition, not just a preference.

## Stretch

- Redo task 3–5 for a **2-regional-DC** split instead of 4 (East/West), and compare all three designs (1, 2, 4 DCs) in one small table. Where does the marginal cost of adding *another* DC start outweighing the marginal safety-stock savings from further pooling?
- Crunch Gear's fastest-growing customer segment is willing to pay a $6/order shipping surcharge for 1–2 day delivery. Roughly how many DTC orders per year would Crunch Gear need at that surcharge to fully offset Design B's extra $410,000/year? (State your assumption about total addressable orders.)

## Submission

Commit `networks.md` to your portfolio under `c40-week-01/exercise-03/`.
