# Week 6 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific question comes up.

## Install (if you haven't already, from earlier weeks)

- **PostgreSQL 16+** — this course's primary data engine: <https://www.postgresql.org/download/> · macOS: [Postgres.app](https://postgresapp.com/) is the easiest. Linux: `sudo apt install postgresql` / `sudo dnf install postgresql-server`. Windows: the EDB installer.
- **SQLite 3.35+** — the zero-setup fallback; ships on macOS and most Linux already: <https://www.sqlite.org/download.html>. Check with `sqlite3 --version`. Note: SQLite has no built-in `STDDEV`/`STDDEV_SAMP` — this week's lead-time-reliability metric needs pandas (or a Postgres session) to compute directly.
- **Python 3.10+ with pandas** — `pip install pandas` (add `psycopg2-binary` if querying Postgres from Python, or `sqlalchemy` for a nicer `read_sql` interface).
- **A GUI (optional)** — [DBeaver](https://dbeaver.io/) (free, both engines) or [pgAdmin](https://www.pgadmin.org/) (Postgres) — handy for eyeballing the `purchase_orders` seed table before you start writing queries against it.
- If SQL syntax (joins, `GROUP BY`, window functions) feels shaky going into this week, [C33 Crunch SQL](../../../C33-CRUNCH-SQL/) weeks 1–4 are the ideal companion reading — same two engines, same core vocabulary this week assumes.

## Required reading (this week's core)

- **CIPS (Chartered Institute of Procurement & Supply) — "What is Spend Analysis?":** <https://www.cips.org/intelligence-hub/procurement/spend-analysis>
  *Why: a practitioner-level overview of spend analysis, classification, and taxonomy — the discipline behind Lecture 1's spend cube.*
- **ISM (Institute for Supply Management) — Glossary of Key Supply Chain Terms:** <https://www.ismworld.org/supply-management-news-and-reports/news-publications/inside-supply-management-magazine/glossary-of-key-supply-chain-terms/>
  *Why: search "tail spend," "maverick buying," "total cost of ownership," and "safety stock" — the standard industry definitions every lecture this week follows.*
- **Gartner — "Total Cost of Ownership (TCO)"** (search the IT glossary): <https://www.gartner.com/en/information-technology/glossary>
  *Why: the original, widely cited TCO framework — IT-focused in origin, but the same "look past the sticker price" logic Lecture 3 applies to suppliers.*
- **ASCM Supply Chain Dictionary — "Safety Stock":** <https://www.ascm.org/learning-development/certifications-credentials/scmdictionary/>
  *Why: the same z-score/service-level safety-stock formula from Week 5, reapplied here to supply-side (lead-time) variability instead of demand-side variability.*

## Reference (keep in tabs)

- **PostgreSQL — Aggregate Functions, including `STDDEV_SAMP`:** <https://www.postgresql.org/docs/current/functions-aggregate.html>
- **PostgreSQL — `GROUPING SETS`, `CUBE`, and `ROLLUP`:** <https://www.postgresql.org/docs/current/queries-table-expressions.html#QUERIES-GROUPING-SETS>
  *Why: the real multi-dimensional spend-cube syntax used in Exercise 1.*
- **PostgreSQL — Window Functions:** <https://www.postgresql.org/docs/current/tutorial-window.html>
  *Why: the `SUM(...) OVER (...)` pattern behind every cumulative-percentage/Pareto calculation this week.*
- **pandas — `groupby` and `.std()`:** <https://pandas.pydata.org/docs/reference/api/pandas.core.groupby.GroupBy.std.html>
- **Investopedia — "Normalization":** <https://www.investopedia.com/terms/n/normalization.asp>
  *Why: a finance-adjacent explainer of min-max scaling, the technique behind every scorecard this week.*

## Practice beyond this week's dataset

- **PostgreSQL Exercises** — free, browser-based, graded `SELECT`/`GROUP BY`/window-function drills: <https://pgexercises.com/>
- **Kaggle — "procurement" / "supply chain" dataset search:** <https://www.kaggle.com/search?q=procurement+in%3Adatasets> — real (if messier) public purchase-order and supplier datasets, good for stretching beyond the course's own seed data once this week feels solid.
- **ASCM's SCOR model, "Source" process overview:** <https://www.ascm.org/learning-development/certifications-credentials/scor-p/> — the industry-standard framework; this week lives entirely inside SCOR's "Source" process (as opposed to Plan/Make/Deliver/Return covered in other weeks).

## Deeper background (optional this week)

- **Monczka, R., Handfield, R., et al., *Purchasing and Supply Chain Management*** — the standard graduate/professional textbook covering spend analysis, supplier scorecards, and TCO in far more depth than one week can; most university libraries carry it or an earlier edition.
- **CAPS Research (Center for Advanced Procurement Strategy) — published research briefs:** <https://www.capsresearch.org/publications> — free procurement-specific research, including benchmark data on supplier scorecarding and should-cost practices across industries.

## Glossary

| Term | Definition |
|------|------------|
| **Spend cube** | Purchase data structured to be sliced along multiple dimensions (category, supplier, business unit, time) at once. |
| **Category taxonomy** | The classification scheme used to group similar spend, usually 2–3 levels deep. |
| **Pareto / 80-20 analysis** | Ranking spend (or any metric) to find the small subset of suppliers/items driving most of the total. |
| **Tail spend** | The long list of small, fragmented purchases spread across many vendors, individually too small to justify a negotiated contract. |
| **Maverick buying** | Purchases made off-contract, bypassing negotiated agreements and preferred-supplier lists. |
| **Supplier scorecard** | A weighted score combining cost, quality, delivery, and reliability metrics into one comparable number per supplier. |
| **Min-max normalization** | Rescaling a metric to a common 0–100 range so metrics in different units can be combined. |
| **OTD (On-Time Delivery)** | The percent of orders received at or before the promised date. |
| **Lead-time reliability** | The consistency (inverse of standard deviation) of a supplier's actual lead time, order to order. |
| **TCO (Total Cost of Ownership)** | Unit price plus freight, quality/rework cost, and inventory-carrying cost — the true cost of a purchase beyond its sticker price. |
| **Safety stock** | Buffer inventory held to protect against demand or supply (lead-time) variability, computed as `z × σ × demand-or-lead-time factor`. |
| **Carrying cost rate** | The annual cost (capital, storage, insurance, obsolescence) of holding a unit of inventory, expressed as a percent of its value. |
| **Single-sourcing** | Awarding all of a category's volume to one supplier — maximizes leverage, concentrates risk. |
| **Dual-sourcing** | Splitting a category's volume across two qualified suppliers — sacrifices some price leverage, buys supply continuity. |
| **Should-cost model** | A bottom-up cost estimate (labor + overhead + SG&A + margin) built independent of any supplier's quote, used to find a negotiation range. |
| **Price variance** | The dollar difference between actual price paid and a standard/contracted cost, per unit or in total. |
| **CMT (Cut-Make-Trim)** | An apparel sourcing arrangement where the brand supplies fabric and the contract manufacturer is paid only for labor to cut, sew, and finish the garment. |

---

*Broken link? Open an issue or PR.*
