# Week 1 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific question comes up.

## Install (needed starting the mini-project, and every week after)

- **PostgreSQL 16+** — this course's primary data engine from Week 2 onward:
  <https://www.postgresql.org/download/> · macOS: [Postgres.app](https://postgresapp.com/) is the easiest. Linux: `sudo apt install postgresql` / `sudo dnf install postgresql-server`. Windows: the EDB installer.
- **SQLite 3.35+** — the zero-setup fallback; ships on macOS and most Linux already: <https://www.sqlite.org/download.html>. Check with `sqlite3 --version`.
- **Python 3.10+ with pandas** — `pip install pandas` (add `psycopg2-binary` if you'll query Postgres from Python, or `sqlalchemy` for a nicer `read_sql` interface).
- **A GUI (optional, later)** — [DBeaver](https://dbeaver.io/) (free, both engines) or [pgAdmin](https://www.pgadmin.org/) (Postgres). Not needed this week; useful once your datasets grow past Week 2.
- If you haven't taken [C33 Crunch SQL](../../C33-CRUNCH-SQL/) yet and SQL syntax feels unfamiliar in the mini-project, its Week 1 is the ideal companion reading — same engines, same `SELECT`/`WHERE`/`GROUP BY` you'll need here.

## Required reading (this week's core)

- **ASCM Supply Chain Dictionary** — the industry's canonical glossary of every term this week uses (node, echelon, OTIF, fill rate, cross-dock, and more): <https://www.ascm.org/learning-development/certifications-credentials/scmdictionary/>
  *Why: this is the vocabulary you'll be tested against for the rest of the course — and in any real operations job.*
- **CSCMP — Supply Chain Management Definitions and Glossary:** <https://cscmp.org/CSCMP/Educate/SCM_Definitions_and_Glossary_of_Terms.aspx>
  *Why: a second major industry body's definitions, useful for cross-checking ASCM's — the two mostly agree, and where they don't is itself instructive.*
- **Lee, Padmanabhan & Whang (1997), "The Bullwhip Effect in Supply Chains,"** *Sloan Management Review*: <https://sloanreview.mit.edu/article/the-bullwhip-effect-in-supply-chains/>
  *Why: the paper that named the effect and laid out its four causes — read it once now, come back to it in Weeks 3–4 and 10.*
- **Investopedia — "Cash Conversion Cycle":** <https://www.investopedia.com/terms/c/cashconversioncycle.asp>
  *Why: the finance-side explanation of the identical C2C formula from Lecture 2 — useful for seeing how operations and finance describe the same number differently.*

## Reference (keep in tabs)

- **PostgreSQL — "Queries" (you'll need this from Week 2):** <https://www.postgresql.org/docs/current/queries.html>
- **PostgreSQL — Aggregate Functions** (`SUM`, `AVG`, `COUNT`, `GROUP BY` — you'll use these constantly to compute KPIs from real tables): <https://www.postgresql.org/docs/current/functions-aggregate.html>
- **pandas — "10 minutes to pandas"** (official quickstart): <https://pandas.pydata.org/docs/user_guide/10min.html>
  *Why: the fastest on-ramp if pandas is new to you before the mini-project.*
- **Eppen, G.D. (1979), "Effects of Centralization on Expected Costs in a Multi-Location Newsboy Problem,"** *Management Science*: <https://pubsonline.informs.org/doi/10.1287/mnsc.25.5.498>
  *Why: the formal derivation of the square root law referenced in Lecture 3 — go here if you want the real math, not just the intuition.*
- **MIT Center for Transportation & Logistics — research library:** <https://ctl.mit.edu/research>
  *Why: free working papers on network design, facility location, and resilience — browse for anything matching this week's topics.*

## Practice beyond this week's datasets

- **ASCM's SCOR (Supply Chain Operations Reference) model overview:** <https://www.ascm.org/learning-development/certifications-credentials/scor-p/> — the industry-standard framework for the exact processes (Plan/Source/Make/Deliver/Return) this course covers week by week; skim the framework diagram to see how Week 1's foundations map onto it.
- **PostgreSQL Exercises** — free, browser-based, graded `SELECT` drills, useful warm-up before the mini-project if SQL is rusty: <https://pgexercises.com/>
- **Kaggle — "Supply Chain" dataset search:** <https://www.kaggle.com/search?q=supply+chain+in%3Adatasets> — real (if messy) public datasets, good for later weeks when you want practice beyond the course's own seed data.

## Deeper background (optional this week)

- **Chopra, S. & Meindl, P., *Supply Chain Management: Strategy, Planning, and Operation*** — the standard graduate textbook covering everything in this week's lectures in far more depth; most university libraries carry it or an earlier edition.
- **Forrester, J.W. (1958), "Industrial Dynamics: A Major Breakthrough for Decision Makers,"** *Harvard Business Review*: <https://hbr.org/1958/07/industrial-dynamics-a-major-breakthrough-for-decision-makers>
  *Why: the origin of systems-dynamics thinking that the bullwhip effect was later built on.*

## Glossary

| Term | Definition |
|------|------------|
| **Node** | Any point in the chain — supplier, plant, DC, retailer, customer. |
| **Echelon** | A tier/stage the product physically passes through and stops at. |
| **Lane** | A transportation link between two nodes, with its own cost, lead time, and mode. |
| **Push** | Work driven by forecast, before a real customer order exists. |
| **Pull** | Work driven by an actual customer order. |
| **Decoupling point** | Where the chain switches from push to pull. |
| **Bullwhip effect** | Small consumer-demand fluctuations amplifying into large order swings further upstream. |
| **OTIF** | On-Time In-Full — the fraction of orders both delivered on time and shipped complete. |
| **Fill rate** | Completeness of shipment vs. order, measured at the unit, order, or line level. |
| **Perfect order** | On-time AND in-full AND damage-free AND accurately invoiced, all required. |
| **Inventory turns** | COGS ÷ Average Inventory Value — how many times inventory is sold and replaced per period. |
| **DIO** | Days Inventory Outstanding — 365 ÷ inventory turns. |
| **DSO** | Days Sales Outstanding — average days to collect cash after a sale. |
| **DPO** | Days Payable Outstanding — average days before the company pays its own suppliers. |
| **Cash-to-cash cycle (C2C)** | DIO + DSO − DPO — days company cash stays tied up in the operating cycle. |
| **Centralized network** | One (or few) large facility serving all demand. |
| **Distributed network** | Several smaller regional facilities, each serving a zone. |
| **Square root law** | Total safety stock across n pooled locations scales roughly with √n, not n. |
| **Cross-dock** | A facility that reloads inbound freight to outbound trucks same-day, without storing inventory. |
| **Total landed cost** | Inventory holding + transportation + facility + service-failure cost, compared across network designs. |

---

*Broken link? Open an issue or PR.*
