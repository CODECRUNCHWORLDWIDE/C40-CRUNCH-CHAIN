# Challenge 1 — Build an OTIF Report Query

**Time:** ~90 minutes. **Difficulty:** Medium-hard. **One correct set of numbers, many valid ways to write the SQL.**

## The scenario

Crunch Gear's VP of Operations saw Lecture 2's single OTIF number (58.3%) and immediately asked the question every real manager asks the moment they see one aggregate number: *"Okay, but which region and which carrier is dragging that down?"* Your job is to build the query pack that answers it — fully, correctly, in one sitting, from data that's messier than Lecture 2's clean walkthrough (this dataset has short-shipments, late deliveries, a damaged-in-transit flag, and an invoice-accuracy flag, same as Week 1's mini-project data).

## Your task

Build a report — one `.sql` file, several queries, or one query with multiple result sets, your choice — that answers all of the following, computed at the **order level** (an order counts only if *every* line and *every* shipment it took satisfies the condition):

1. **Overall OTIF%** across all 12 orders.
2. **OTIF% by region** (Northeast, Southeast, Midwest, West).
3. **OTIF% by carrier.**
4. **Overall unit fill rate, order fill rate, and perfect order rate.** Perfect order requires on-time **and** in-full **and** damage-free (`damaged_in_transit = FALSE` on every line) **and** invoice-accurate (`invoice_accurate = TRUE` on every line) — all four, same strict definition as Week 1's mini-project.
5. **A one-paragraph write-up** comparing your answers to #1 and #4's perfect-order number: in this dataset, are they the same or different? Explain *why*, referencing which specific orders/lines carry the damage and invoice-accuracy flags — don't just report the numbers, explain the relationship between them.

## Constraints

- Compute everything at the **order** level, not the line or shipment level — an order with two shipments, one on time and one late, is a late order, full stop, same as Exercise 2 Task 5 and Lecture 2 Section 2 modeled it.
- No fan-out bugs: if any of your numbers look implausibly high or low, suspect a join multiplying rows before you suspect the data. Cross-check with `COUNT(DISTINCT order_id)` at key points in your queries while you're debugging, even if you remove that debug line from your final answer.
- SQL only for this challenge — no pandas. (Challenge 2 is where pandas comes back in.)
- Show your SQL, not just a results table — a correct number backed by SQL you can't produce on request is not a passing answer here or in a real job.

## Hints

<details>
<summary>On computing "in full" per order when an order has multiple lines</summary>

You need the **sum** of `qty_shipped` across every line on the order to be `>=` the **sum** of `qty_ordered` across every line on that same order — not "every individual line happened to ship in full." Watch out: those two conditions usually agree, but they're not logically the same thing, and a report that silently swapped one for the other would misjudge an order where one line over-shipped and another under-shipped by a compensating amount. State in your write-up which of the two you used and why you picked it.

</details>

<details>
<summary>On perfect order's damage/invoice check spanning multiple lines</summary>

An order is damage-free only if **none** of its shipment lines have `damaged_in_transit = TRUE` — one bad line fails the whole order, same logic as Week 1 Lecture 2's "perfect order requires every dimension" rule. A `NOT EXISTS` or a `bool_and()`/`MIN()` aggregate (Postgres: `bool_and(NOT damaged_in_transit)`; portable: `MIN(CASE WHEN damaged_in_transit THEN 0 ELSE 1 END) = 1`) both work — pick whichever you can explain clearly.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Correctness | Numbers don't match a careful hand-check against the seed data | All five report sections match the underlying data exactly |
| Fan-out safety | A `SUM()` inflated by an unnoticed join multiplication | Every aggregate is provably at the correct grain |
| SQL structure | Five-deep nested subqueries, hard to follow | Clear CTEs, each one named for what it represents |
| The write-up (#5) | Restates the two numbers with no explanation | Names the specific order(s)/line(s) responsible for the gap (or the lack of one) and reasons about it |

## Submission

Commit `challenge-01.sql` (plus your write-up, inline as SQL comments or a separate `challenge-01.md`) to your portfolio under `c40-week-02/challenge-01/`.
