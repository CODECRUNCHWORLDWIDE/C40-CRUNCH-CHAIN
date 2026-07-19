# Challenge 2 — Reconcile SQL and pandas KPIs

**Time:** ~90 minutes. **Difficulty:** Medium-hard. **The bug is real, planted, and will not look obviously wrong.**

## The scenario

A teammate computed this month's unit fill rate two ways: once in SQL (the way Lecture 2 taught it), once in pandas (the way Lecture 3 taught it). The SQL answer is **98.4%**. The pandas answer, from the script below, comes out to **76.7%**. Both numbers *look* plausible on their own — 76.7% isn't an impossible fill rate, it's just a concerning one. That's exactly what makes this bug dangerous: nothing about the wrong number screams "recheck me." Only comparing it against an independent SQL calculation catches it at all — which is the entire reason Lecture 3 insisted on computing things two ways in the first place.

Your job: find the bug, fix it, and explain — in writing — precisely why it produced a number that's wrong but not obviously wrong.

## The buggy script

```python
import pandas as pd
import psycopg2  # or sqlite3 -- either works with the equivalent read_sql calls

conn = psycopg2.connect(dbname="crunch_chain", host="localhost")

order_lines_df    = pd.read_sql("SELECT * FROM order_lines", conn)
orders_df         = pd.read_sql("SELECT order_id, promised_date FROM orders", conn)
shipments_df      = pd.read_sql("SELECT shipment_id, order_id, delivery_date FROM shipments", conn)
shipment_lines_df = pd.read_sql("SELECT order_line_id, shipment_id, qty_shipped FROM shipment_lines", conn)
conn.close()

merged = (
    order_lines_df
    .merge(orders_df, on="order_id")
    .merge(shipments_df, on="order_id")   # <-- merges shipments onto every order LINE by order_id
    .merge(shipment_lines_df, on=["order_line_id", "shipment_id"], how="left")
)
merged["qty_shipped"] = merged["qty_shipped"].fillna(0)

fill_rate = merged["qty_shipped"].sum() / merged["qty_ordered"].sum()
print(f"Unit fill rate: {fill_rate:.1%}")   # prints 76.7%
```

## Your task

1. **Reproduce the bug.** Run the script (or trace through it by hand using Exercise 1's seed data) and confirm you get 76.7%, not 98.4%.
2. **Find the exact row-level cause.** Print (or hand-trace) `len(merged)` — it will not be 18, the number of order lines. Figure out exactly which order(s) are responsible for the extra rows, and why. *(Hint: it's the same two orders Exercise 2 Task 4 used to demonstrate a SQL fan-out — this is the identical bug, in pandas instead of SQL.)*
3. **Explain, precisely, why `qty_shipped`'s sum came out correct (1,495 — matching SQL) while `qty_ordered`'s sum did not.** This is the subtle part: one column in the merged DataFrame is right and the other is wrong, from the *same* buggy merge. Walk through why.
4. **Fix it.** Rewrite the pandas code so it merges `shipments` in by the correct key — through `shipment_lines`, not directly by `order_id` — and confirm you now get 98.4%, matching SQL exactly.
5. **Write up your findings** in `challenge-02.md`: the wrong number, the row-level cause, why one column was right and one was wrong, your fix, and the corrected number.

## Constraints

- Don't just guess-and-check merge keys until the number looks right — that's how the bug got introduced in the first place. Actually inspect `merged` (`.shape`, `merged[merged.order_id == 3]`, `.groupby("order_id").size()`) and reason from what you see.
- Your fix must still use `orders_df` and `shipments_df` somewhere if your final report needs their columns (`promised_date`, `delivery_date`) — the fix is about the merge *key*, not about dropping tables from the pipeline.
- State both numbers (76.7% and 98.4%) explicitly in your write-up, plus the SQL query you used as ground truth to catch the discrepancy in the first place.

## Hints

<details>
<summary>On why <code>qty_shipped</code>'s sum was accidentally correct</summary>

The second merge, onto `shipment_lines`, uses the **composite key** `["order_line_id", "shipment_id"]` — and each `(order_line_id, shipment_id)` pair really does only match one true shipment line. So even though the first merge created extra, spurious `(order_line_id, shipment_id)` combinations that never happened in reality, the second merge correctly finds no match for the spurious ones (`NaN` → filled to `0`) and the one correct match for the real one. `qty_shipped` self-corrects. `qty_ordered`, though, was already attached to every order line **before** the fan-out merge ever happened, so it gets carried along and duplicated on every spurious extra row, with nothing downstream to catch it.

</details>

<details>
<summary>On the fix</summary>

Don't merge `shipments` onto `order_lines` by `order_id` at all. Merge `shipment_lines` onto `order_lines` first (by `order_line_id` — a true one-to-one relationship in this dataset), *then* merge `shipments` onto that result by `shipment_id` (also one-to-one — one `shipment_line` belongs to exactly one `shipment`). Every merge step should be justified by asking "is this actually a many-to-one or one-to-one relationship on this key?" — `order_id` is not the right key to reach `shipments` from `order_lines`, because one order can have more than one shipment.

</details>

## How success is judged

| Signal | Weak answer | Strong answer |
|--------|-------------|----------------|
| Root cause | "The merge was wrong" | Names the exact orders, the exact extra row count, and the exact mechanism |
| The asymmetry (#3) | Skipped or hand-waved | Correctly explains why one column self-corrected and the other didn't |
| Fix | Numbers now match, but by trial and error | Fix follows directly from the stated root cause, and you can explain why it's *structurally* correct, not just numerically lucky |
| Habit | Treats this as a one-off bug | Explicitly connects it back to Lecture 2 Section 1's SQL fan-out warning and Lecture 3 Section 4's merge trap — same failure mode, different tool |

## Submission

Commit `challenge-02.py` (or `.ipynb`) and `challenge-02.md` to your portfolio under `c40-week-02/challenge-02/`.
