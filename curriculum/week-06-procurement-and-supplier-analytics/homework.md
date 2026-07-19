# Week 6 — Homework

Five problems, ~4.5 hours total, spread across the week. These reinforce the lectures with a mix of SQL/pandas computation, written reasoning, and one small research task. Commit each.

All formulas follow this week's lectures unless a problem says otherwise.

---

## Problem 1 — Twenty warm-up questions (55 min)

Short answers, one or two sentences each, in `warmups.md`. These check the week's vocabulary is solid before you lean on it in the challenges and mini-project.

1. What's the difference between a spend category and a spend cube?
2. Write, in words, what `SUM(...) OVER ()` computes that a plain `SUM(...) GROUP BY` cannot, in the same query.
3. What does the Pareto/80-20 view of supplier spend tell you that a flat list sorted by dollar amount doesn't?
4. Define "tail spend" in your own words.
5. Define "maverick buying," and explain how it's different from tail spend (they overlap but aren't identical).
6. Name the four components of a standard supplier scorecard used in this week's lectures.
7. Why can't you average a percentage, a dollar figure, and a day-count directly into one score?
8. What does min-max normalization do, and why does the formula flip direction for "lower is better" metrics?
9. What's the difference between on-time delivery (OTD) rate and lead-time reliability (standard deviation of lead time)? Give an example where a supplier could score well on one and poorly on the other.
10. Why should a supplier's scorecard report its sample size (`n_orders`) alongside its score?
11. Name the four components of the TCO model from Lecture 3.
12. Write the safety-stock formula from Lecture 3 from memory, and explain what each symbol means.
13. Why does *lead-time variability*, not average lead time, drive the safety-stock calculation?
14. What is an annual carrying-cost rate, and roughly what range is typical in apparel?
15. What's the difference between "TCO says Supplier A is cheaper" and "we should single-source with Supplier A"?
16. Name one risk of single-sourcing and one cost of dual-sourcing.
17. What is a should-cost model, and how is it different from a supplier's quoted price?
18. Why is a should-cost model deliberately conservative — i.e., why shouldn't a 200%+ gap between should-cost and actual price be taken at face value?
19. Define "price variance," and state which sign convention (positive/negative) this week used for "unfavorable."
20. Why should a price-variance report never stop at the net total across all suppliers?

---

## Problem 2 — Score a new set of suppliers (75 min)

A new quarter (Q2 2026), a different category: **Trims**, three suppliers, one of them brand new to Crunch Gear. Load this into a fresh table (or a new `quarter = 'Q2'` column on the existing schema) and compute the full scorecard yourself — this data has not appeared anywhere else this week.

```sql
CREATE TABLE trims_q2 (
    po_id           INTEGER PRIMARY KEY,
    supplier        TEXT    NOT NULL,
    order_date      DATE    NOT NULL,
    promised_date   DATE    NOT NULL,
    receipt_date    DATE    NOT NULL,
    qty_ordered     INTEGER NOT NULL,
    qty_received    INTEGER NOT NULL,
    unit_price      NUMERIC NOT NULL,
    defect_qty      INTEGER NOT NULL,
    freight_cost    NUMERIC NOT NULL
);

INSERT INTO trims_q2 VALUES
(101,'ButtonWorks Supply','2026-04-04','2026-04-18','2026-04-17',22000,22000,0.43,90,350),
(102,'ButtonWorks Supply','2026-04-25','2026-05-09','2026-05-12',19000,18800,0.43,140,320),
(103,'ButtonWorks Supply','2026-05-20','2026-06-03','2026-06-02',17000,17000,0.44,60,300),
(104,'Zephyr Hardware Co.','2026-04-08','2026-04-22','2026-04-29',16000,15500,0.40,220,310),
(105,'Zephyr Hardware Co.','2026-05-05','2026-05-19','2026-05-25',17000,16600,0.40,280,330),
(106,'Zephyr Hardware Co.','2026-06-01','2026-06-15','2026-06-14',15000,14900,0.41,150,290),
(107,'SwiftSnap Fasteners','2026-04-12','2026-04-26','2026-04-25',12000,12000,0.46,30,260),
(108,'SwiftSnap Fasteners','2026-05-15','2026-05-29','2026-05-28',13000,12950,0.46,45,280);
```

In `trims-scorecard.sql` or `.py`:

1. Compute total spend, and each supplier's share of it.
2. Compute the four scorecard metrics per supplier (volume-weighted price, defect rate %, OTD %, lead-time std) and report `n_orders` for each.
3. Normalize and combine into a weighted score using this week's default weights (30/30/25/15).
4. **SwiftSnap Fasteners is a brand-new supplier with only 2 orders and the best raw numbers of the three.** In 3–4 sentences, state whether you'd recommend expanding their volume next quarter and what evidence would change your answer.

---

## Problem 3 — Explain the analytics gap (35 min)

In `analytics-writeup.md`, answer in prose (no more than 400 words total):

1. Explain, to someone who has never done procurement analytics, why a supplier with the *lowest unit price* in a category can still be the *most expensive* choice once TCO is computed. Use a concrete (invented but plausible) example outside of Crunch Gear.
2. Explain why a scorecard's weighted formula (e.g., 30% cost / 30% quality / 25% OTD / 15% lead-time) is a business judgment, not a mathematical fact — what would change if a company weighted quality at 50% instead of 30%, and in what kind of business might that reweighting make sense?
3. In your own words: why is reporting only the *net* price variance across all suppliers potentially misleading, even when the net number itself is completely accurate?

---

## Problem 4 — Research one real should-cost or supplier-scoring story (55 min)

Find a real, findable news article, case study, or company report describing a should-cost initiative, a supplier-scorecard program, or a major sourcing/dual-sourcing decision (search something like "[company] should-cost model," "[company] supplier scorecard," or "[company] dual sourcing supply chain"). In `research.md`:

1. Cite the source (name, publication, rough date).
2. Summarize the situation in 2–3 sentences, in your own words (do not paste large quotes — see the note below).
3. Using this week's vocabulary, classify what the company was doing: spend consolidation, supplier scoring, TCO/should-cost analysis, single- vs. multi-sourcing, or some combination.
4. Speculate, using this week's frameworks, what trade-off the company was likely weighing (e.g., cost vs. concentration risk, price vs. quality) and what you'd want to see in their data to confirm it.

**A note on sources:** summarize in your own words. If you quote a sentence directly, keep it short and attribute it clearly.

---

## Problem 5 — Extend the week's dataset with a new supplier (70 min)

Working from this week's `purchase_orders` seed (or a copy of it), extend your understanding of how a scorecard reacts to new information:

1. Add **4 new rows** to `purchase_orders` for a brand-new CMT supplier of your invention (pick a name, a plausible price near the existing CMT range of $15–23, and vary the quality/delivery performance across the 4 orders — at least one late, at least one with defects).
2. Recompute the CMT scorecard from Lecture 2 with your new supplier included (4 suppliers now, not 3).
3. Write 2–3 sentences: did adding a 4th supplier change any of the *other three* suppliers' normalized scores, even though their raw metrics didn't change? Explain why (or why not), tying back to how min-max normalization works.

**Deliver** `extend.sql` or `extend.py` (your additions + recomputed scorecard) plus the 2–3 sentence reflection, in the same file or a short `extend-notes.md`.

---

## Time budget

| Problem | Time |
|--------:|----:|
| 1 | 55 min |
| 2 | 75 min |
| 3 | 35 min |
| 4 | 55 min |
| 5 | 70 min |
| **Total** | **~4.8 h** |

After homework, take the [quiz](./quiz.md) and ship the [mini-project](./mini-project/README.md).
