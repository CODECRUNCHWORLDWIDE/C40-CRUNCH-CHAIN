# Week 5 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific question comes up.

## Install first

- **PostgreSQL 16+** — the course's primary engine:
  <https://www.postgresql.org/download/> · macOS: [Postgres.app](https://postgresapp.com/) is the easiest. Linux: `sudo apt install postgresql` / `sudo dnf install postgresql-server`. Windows: the EDB installer.
- **SQLite 3.35+** — the zero-setup fallback; ships on macOS and most Linux already: <https://www.sqlite.org/download.html>. Check with `sqlite3 --version`.
- **Python 3.10+ with pandas, numpy, and scipy:**
  ```bash
  pip install pandas numpy scipy sqlalchemy psycopg2-binary
  ```
  `scipy.stats.norm` is required this week for `.ppf()` (z-score lookup) and `.pdf()`/`.cdf()` (fill-rate calculations) — it's the one new dependency vs. earlier weeks.
- **A GUI (optional)** — [DBeaver](https://dbeaver.io/) (free, both engines) or [pgAdmin](https://www.pgadmin.org/) (Postgres).

## Required reading (this week's core)

- **Wikipedia — Economic Order Quantity** (clear derivation, worth reading alongside Lecture 1): <https://en.wikipedia.org/wiki/Economic_order_quantity>
  *Why: a second, independent walk-through of the same derivation, useful if the calculus in Lecture 1 didn't click the first time.*
- **Wikipedia — Newsvendor model:** <https://en.wikipedia.org/wiki/Newsvendor_model>
  *Why: the full critical-ratio derivation, which Lecture 3 deliberately skipped in favor of the result — read this if you want to see where `Q* = F⁻¹(CR)` actually comes from.*
- **SciPy — `scipy.stats.norm` reference:** <https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.norm.html>
  *Why: `.ppf()`, `.pdf()`, and `.cdf()` are the three methods you'll call by name throughout this week's exercises and the mini-project.*
- **PostgreSQL — Mathematical functions (`sqrt`, `power`):** <https://www.postgresql.org/docs/current/functions-math.html>
  *Why: the exact signatures for the square-root and power functions used in every SQL query this week.*

## Reference (keep in tabs)

- **APICS/ASCM — CSCP body of knowledge (inventory management section):** <https://www.ascm.org/learning-development/certifications-credentials/cscp/>
  *Why: the industry-standard certification body's treatment of everything in this week — service level definitions, review policies, safety stock — is worth skimming even if you're not pursuing the certification.*
- **NIST/SEMATECH e-Handbook of Statistical Methods — Normal distribution:** <https://www.itl.nist.gov/div898/handbook/eda/section3/eda3661.htm>
  *Why: a free, rigorous reference on the normal distribution and its quantile function if the z-score mechanics in Lecture 2 feel shaky.*
- **PostgreSQL — `GENERATE_SERIES`** (useful for building synthetic daily-demand tables to simulate against in SQL instead of pandas): <https://www.postgresql.org/docs/current/functions-srf.html>
  *Why: an alternative to Exercise 3's pandas/Python simulation if you'd rather push the day-by-day loop into SQL.*
- **NumPy — `numpy.random.Generator`:** <https://numpy.org/doc/stable/reference/random/generator.html>
  *Why: the modern, recommended way to generate reproducible random numbers in pandas-based simulations (an alternative to the standard-library `random` module used in Exercise 3).*

## Practice beyond this week's catalog

- **MIT OpenCourseWare — Supply Chain Management (search current course listings):** <https://ocw.mit.edu/search/?q=supply%20chain>
  *Why: free graduate-level treatment of inventory theory, if you want to go deeper than this course's applied approach.*
- **APICS/ASCM sample CSCP practice questions:** <https://www.ascm.org/>
  *Why: more multiple-choice reps on the exact vocabulary (CSL, fill rate, EOQ, ROP) this week's quiz draws on.*

## Deeper background (optional this week)

- **Harris, F.W. (1913), "How Many Parts to Make at Once"** — the original EOQ paper (Factory, The Magazine of Management):
  a short historical read available via most university library databases; search the title.
  *Why: EOQ is over a century old and still the industry default — worth seeing where it started.*
- **Silver, Pyke & Peterson, "Inventory Management and Production Planning and Scheduling"** — the classic academic reference text (library/purchase, not free) covering (s,Q), (R,S), and multi-echelon models in full mathematical depth.
  *Why: if you end up doing inventory optimization professionally, this is the standard reference the field cites.*

## Glossary

| Term | Definition |
|------|------------|
| **Holding cost (carrying cost)** | The annual cost of owning one unit of inventory — capital, storage, insurance, obsolescence, shrinkage. Usually expressed as `H = i * C`. |
| **Ordering cost (setup cost)** | The fixed cost of placing one purchase order, independent of order size. |
| **Shortage cost (stockout cost)** | The cost of not having a unit when demand arrives — lost sale, backorder expediting, goodwill loss. |
| **EOQ** | Economic Order Quantity — the order size that minimizes the sum of holding and ordering cost: `sqrt(2DS/H)`. |
| **Cycle-service level (CSL)** | The probability of *not* stocking out during one replenishment cycle — an event probability, blind to shortage size. |
| **Fill rate** | The fraction of total demand *units* satisfied immediately from stock — a magnitude-weighted measure, usually higher than CSL for the same policy. |
| **Demand during lead time (DLT)** | The random total demand that accumulates during the wait between placing and receiving a replenishment order. |
| **Safety stock** | The buffer inventory held above expected demand during lead time, sized as `z * σ_DLT` for a target service level. |
| **Reorder point (ROP)** | The inventory position that triggers a new order: `d̄*L + SS`. |
| **(s,Q) policy** | Continuous review: reorder a fixed quantity `Q` the moment inventory position hits `s`. |
| **(R,S) policy** | Periodic review: every `R` time units, order up to a target level `S`. |
| **Base-stock policy (S-1,S)** | The limiting case of (R,S) with continuous, unit-by-unit review — order one unit the moment one unit sells. |
| **Newsvendor model** | The single-period stocking model for perishable or one-shot products, solved via the critical ratio `CR = Cu/(Cu+Co)`. |
| **Critical ratio (CR)** | In the newsvendor model, the quantile of the demand distribution the optimal order quantity should target. |
| **Risk pooling** | The statistical benefit of aggregating independent demand streams — combined variance grows slower than a simple sum, reducing total safety stock needed for the same service level (subject to the lead-time-variability caveat in Challenge 2). |

---

*Broken link? Open an issue or PR.*
