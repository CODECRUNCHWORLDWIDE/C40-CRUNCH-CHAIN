# Week 1 — Homework

Five problems, ~4.5 hours total, spread across the week. These reinforce the lectures with a mix of hand-computation, written reasoning, and one small research task. Commit each.

All KPI computations use the formulas from Lecture 2 unless a problem says otherwise. Diagrams can be hand-drawn (photographed) or ASCII — clarity matters more than tooling.

---

## Problem 1 — Twenty warm-up questions (60 min)

Short answers, one or two sentences each, in `warmups.md`. These check that Lecture 1–2 vocabulary is solid before you lean on it all course.

1. Define "echelon" in your own words.
2. Name the three flows in a supply chain and the general direction each one moves.
3. What is a decoupling point?
4. Which of the three flows is *fastest* to observe changing, typically — product, information, or cash? Why?
5. What does "landed cost" mean, and why do operations people track it instead of shelf price?
6. Write the OTIF formula from memory.
7. What's the difference between "on-time" measured against ship date vs. delivery date?
8. Write the unit fill rate formula from memory.
9. Give one reason order fill rate and unit fill rate can differ a lot on the same data.
10. What four components make up "perfect order"?
11. Why do perfect order components multiply instead of average?
12. Write the inventory turns formula from memory.
13. What's the relationship between inventory turns and days of inventory (DIO)?
14. Write the cash-to-cash cycle formula from memory, including which term is subtracted.
15. Name one lever a company could pull to reduce DSO.
16. What is the square root law of inventory (one sentence)?
17. Name one advantage and one disadvantage of a centralized network.
18. What is cross-docking, and how is it different from a normal DC?
19. Sketch (describe in words) the shape of the cost-service curve.
20. What are the four components of total landed cost used to compare network designs?

---

## Problem 2 — KPI computation from a new dataset (75 min)

A different Crunch Gear week, 12 orders this time. Compute all five KPIs (treat this dataset as having no damage/invoice issues, so perfect order = OTIF for this problem) and put your work in `warmups2.md` (numerator/denominator shown for each).

| order_id | promised_date | ship_date | qty_ordered | qty_shipped |
|---:|---|---|---:|---:|
| 1 | 2026-05-04 | 2026-05-04 | 90 | 90 |
| 2 | 2026-05-04 | 2026-05-06 | 60 | 60 |
| 3 | 2026-05-05 | 2026-05-05 | 140 | 130 |
| 4 | 2026-05-06 | 2026-05-06 | 75 | 75 |
| 5 | 2026-05-06 | 2026-05-06 | 110 | 100 |
| 6 | 2026-05-07 | 2026-05-09 | 95 | 85 |
| 7 | 2026-05-08 | 2026-05-08 | 200 | 200 |
| 8 | 2026-05-09 | 2026-05-09 | 65 | 65 |
| 9 | 2026-05-10 | 2026-05-11 | 120 | 120 |
| 10 | 2026-05-11 | 2026-05-11 | 80 | 75 |
| 11 | 2026-05-12 | 2026-05-12 | 150 | 150 |
| 12 | 2026-05-13 | 2026-05-13 | 100 | 90 |

Finance snapshot for this problem: annual COGS = $2,100,000; average inventory = $350,000; annual net sales = $3,000,000; average AR = $200,000; average AP = $180,000.

Report: OTIF%, unit fill rate, order fill rate, inventory turns, DIO, DSO, DPO, and C2C.

---

## Problem 3 — Explain the metrics gap (40 min)

In `metrics-writeup.md`, answer in prose (no more than 400 words total):

1. Explain, to someone who has never heard the term, why a business can report "96% fill rate" and still have real customers who are unhappy.
2. Give a realistic (invented but plausible) example, outside of Crunch Gear, of a company whose *unit* fill rate looks great but whose *order* fill rate is mediocre. What kind of product mix would cause that pattern?
3. In your own words: why does a mature operations team track perfect order as its own number instead of just multiplying the components after the fact each time?
4. Name one KPI from this week you think is *most* likely to be gamed or reported misleadingly by a team under pressure to hit a target, and explain how.

---

## Problem 4 — Research one real network decision (60 min)

Pick a real, findable news story or case study (a company opening/closing a distribution center, changing its delivery-speed promise, reshoring/offshoring manufacturing, or similar — a quick web search for "[company name] distribution center opens/closes" or "[company name] supply chain" usually finds one). In `research.md`:

1. Cite the source (name, publication, rough date).
2. Summarize the decision in 2–3 sentences, in your own words (do not paste large quotes — see the note below).
3. Using this week's vocabulary, classify what kind of decision it was: echelon count, centralize vs. distribute, facility role change, or something else.
4. Speculate, using Lecture 3's cost-service framework, on what trade-off the company was likely making — what did they probably gain, and what did it probably cost them?

**A note on sources:** summarize in your own words. If you quote a sentence directly, keep it short and attribute it clearly.

---

## Problem 5 — Extend the mini-project dataset (75 min)

Working from the mini-project's `order_lines` seed (or a copy of it), extend your understanding of the data:

1. Add **5 new rows** to `order_lines` of your own invention: at least one clearly late, one clearly short, one damaged, one with an invoice error, and one that's perfect on all four dimensions. Use a new region if you want ("International") or reuse existing ones.
2. Recompute OTIF, unit fill rate, and perfect order rate with the 5 new rows included (25 total).
3. Write 2–3 sentences on how much each KPI moved, and whether 5 added rows out of 25 should be expected to move a KPI a little or a lot — what does that tell you about how *stable* (or noisy) these metrics are on a small dataset vs. a real company's monthly order volume?

**Deliver** `extend.sql` or `extend.py` (your additions + recomputed KPIs) plus the 2–3 sentence reflection, in the same file or a short `extend-notes.md`.

---

## Time budget

| Problem | Time |
|--------:|----:|
| 1 | 60 min |
| 2 | 75 min |
| 3 | 40 min |
| 4 | 60 min |
| 5 | 75 min |
| **Total** | **~5 h** |

After homework, take the [quiz](./quiz.md) and ship the [mini-project](./mini-project/README.md).
