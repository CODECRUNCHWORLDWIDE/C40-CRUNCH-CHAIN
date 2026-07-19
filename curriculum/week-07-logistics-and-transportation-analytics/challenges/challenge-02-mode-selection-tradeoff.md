# Challenge 2 — Mode-Selection Trade-off

**Time:** ~90 minutes. **Difficulty:** Medium. **No single right answer — judgment is the point.**

## The scenario

Part A looks backward: somewhere in the 124 shipments Crunch Gear already made, a few of them used a mode that, in hindsight, wasn't the right call for the weight and distance involved. Find them and put a dollar figure on the waste. Part B looks forward: six new orders just came in, each with a real deadline and a real cost if it's late. Pick the mode for each and defend it.

## Part A — Audit the mode choices Crunch Gear already made (45 min)

Lecture 1 showed that **FTL is priced per truck, not per pound** — so a *small* shipment sent FTL pays for capacity it never uses, while a *near-full* FTL shipment is a great deal. Similarly, Intermodal only beats FTL's per-mile rate once distance is long enough to absorb the extra drayage/rail-transfer handling.

1. Query every `FTL` shipment where `weight_lbs < 15000` (a truck running at under a third of its ~45,000 lb capacity). List `shipment_id`, `lane_id`, `weight_lbs`, `freight_cost`, and `freight_cost / weight_lbs AS cost_per_lb`.

2. For each shipment you found, estimate what it **would have cost as LTL instead**, using the network-wide average LTL cost per pound from Exercise 1 (Task 8). Compute the difference (`ftl_cost - estimated_ltl_cost`) as your estimate of the waste.

3. Do the same audit in the other direction: find any `Intermodal` shipment with `distance_miles < 600` (Lecture 1 said intermodal rarely pays off below ~500-600 miles because of the extra handling). How many are there, and does the data support or contradict the rule of thumb?

4. Write two or three sentences: **overall, does Crunch Gear's mode selection look disciplined, or is there a real, quantified opportunity here?** Back your claim with a total dollar figure, not just a vibe.

**A caution built into this task on purpose:** the "estimated LTL cost" you compute using the network-wide average is a rough stand-in, not a real quote — LTL rates depend on freight class and the exact weight break a shipment lands in, which you don't have granular data for. State that limitation explicitly in your write-up. A number with an honestly stated margin of error is more valuable than a falsely precise one.

## Part B — Choose the mode for six new orders (45 min)

Set up the new order slate:

```sql
CREATE TABLE pending_orders (
    order_id          INTEGER PRIMARY KEY,
    description       TEXT    NOT NULL,
    lane_id            TEXT    NOT NULL,
    weight_lbs         NUMERIC NOT NULL,
    deadline_days       NUMERIC NOT NULL,   -- must arrive within this many days
    stockout_cost_usd   NUMERIC NOT NULL    -- $ lost if it does NOT arrive in time
);

INSERT INTO pending_orders VALUES
(1, 'Wholesale reorder, regular replenishment',       'AUS-DAL', 6200,  6,  400),
(2, 'DTC flash-sale inventory, hard launch date',       'AUS-DEN',  850,  4, 9500),
(3, 'Fabric restock from mill, no urgency',             'HPH-ATX', 22000, 45, 1200),
(4, 'Retail partner reset, contractual delivery window', 'MEM-CHI', 4100,  3, 6000),
(5, 'DC-to-DC safety-stock top-off',                    'REN-AUS', 15500, 5,  300),
(6, 'Trade-show samples, must not miss the show',       'AUS-ATL',  180,  2, 4000);
```

For each order:

1. **Estimate the cost under 2-3 plausible modes**, using the mode-level average `cost_per_lb` figures from Lecture 1 / Exercise 1 (Task 8) and the mode's typical transit-day range from Lecture 1, Section 1.
2. **Eliminate any mode that can't realistically meet `deadline_days`** given its typical transit range — be honest about variance, not just the best case (a mode whose *typical* transit is 3-7 days is a bad bet against a 3-day hard deadline, even though 3 days is technically within range).
3. **Pick the cheapest mode that survives step 2**, and write one sentence justifying it against the `stockout_cost_usd` — would a cheaper-but-riskier mode have been worth the gamble, given what a miss costs?
4. Total the cost of your six choices, and compare it to two naive baselines: **(a)** send everything by the single cheapest mode available on each lane regardless of deadline, and **(b)** send everything by Air. Report all three totals side by side.

## Constraints

- Use the mode-level `cost_per_lb` and transit-range figures already established in Lecture 1 and Exercise 1 — don't invent new rate assumptions without saying so.
- Every mode choice in Part B needs a **written one-sentence justification** tied explicitly to `deadline_days` and `stockout_cost_usd`. A bare mode name with no reasoning is an incomplete answer, even if it happens to be the "right" one.
- It's fine — expected, even — for reasonable people to pick a different mode than you did on order 2 or order 6, where the deadline is tight and the stockout cost is high. Defend your call; you don't have to match a hidden answer key.

## How success is judged

| Signal | Weak submission | Strong submission |
|---|---|---|
| Part A rigor | Lists FTL shipments with no cost comparison | Computes an estimated waste figure per shipment and a total, with the estimation caveat stated |
| Part B reasoning | Picks the cheapest mode for every order, ignoring deadlines | Explicitly eliminates infeasible modes before picking on cost |
| Justification | No sentence, or a restated fact ("Air is fast") | Ties the choice to the specific `stockout_cost_usd` vs. the price difference between modes |
| Comparison | No baseline comparison | All three totals (your plan, cheapest-always, Air-always) reported and briefly interpreted |

## Submission

Commit `challenge-02.md` (with any SQL/Python you used, inline or attached) to your portfolio under `c40-week-07/challenge-02/`.
