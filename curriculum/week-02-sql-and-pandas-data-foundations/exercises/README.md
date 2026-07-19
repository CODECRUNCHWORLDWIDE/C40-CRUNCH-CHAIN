# Week 2 — Exercises

Three guided exercises. Unlike Week 1's pencil-and-paper exercises, everything here runs against a real PostgreSQL (or SQLite) database — you'll type SQL, run it, and check your results against exact row counts and values.

1. **[Exercise 1 — Seed the network database](exercise-01-seed-the-network-database.md)** — build the full nine-table schema from Lecture 1 and load a month of Crunch Gear order/shipment/inventory data. *(~1.5h)*
2. **[Exercise 2 — Join orders to shipments](exercise-02-join-orders-to-shipments.md)** — build a correct, fan-out-safe fulfillment view across orders, order lines, shipments, and shipment lines. *(~1.5h)*
3. **[Exercise 3 — Rolling inventory with window functions](exercise-03-rolling-inventory-with-windows.md)** — running inventory balances and lead-time percentiles, straight from Lecture 2 Sections 3–4. *(~1.5h)*

## Before you start

- **Do Exercise 1 first, in full, before touching Exercise 2 or 3.** Every later exercise, challenge, and the mini-project all query the exact dataset Exercise 1 loads — if your row counts don't match Exercise 1's verification checks, every later exercise's "expected" numbers will be wrong too, and you'll spend more time debugging a seed mismatch than learning the actual lesson.
- You've read Lecture 1 (schema design) and Lecture 2 (joins/aggregation/windows) — these exercises assume you've seen the query patterns already and are now applying them yourself, not seeing them cold.
- You have `psql` or `sqlite3` open in a terminal, ready to run SQL interactively.

## Suggested workflow

- Run every query as you write it — don't write five queries blind and debug them all at once. SQL errors are cheap to fix one at a time and expensive to fix in a pile.
- When a query's result doesn't match the "expected" hint, don't just tweak numbers until it matches — run the query one join at a time (`SELECT * FROM order_lines ol JOIN orders o ...` with no aggregation yet) and look at the raw rows. The bug is almost always visible in the raw join output before you ever add a `GROUP BY` or `SUM()` on top of it.
- Keep every exercise's SQL in its own `.sql` file with `-- Task N` comments, same as the schema file from Exercise 1 — you'll reuse pieces of this SQL directly in the challenges and mini-project.
