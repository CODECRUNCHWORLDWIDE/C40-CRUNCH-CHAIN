# Week 11 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific question comes up. The full `network_risk_register` (16 rows) and `daily_ops` (180 rows) seed data, referenced from the [week README](./README.md), are at the bottom of this page.

## Install first

- **PostgreSQL 16+** — the course's primary engine: <https://www.postgresql.org/download/> · macOS: [Postgres.app](https://postgresapp.com/). Linux: `sudo apt install postgresql` / `sudo dnf install postgresql-server`. Windows: the EDB installer.
- **SQLite 3.35+** — the zero-setup fallback, ships on macOS and most Linux already: <https://www.sqlite.org/download.html>. Check with `sqlite3 --version`. Note: SQLite has no built-in `STDDEV` — this week's z-score work (Lecture 3, Exercise 3, Challenge 2) runs the standard-deviation step in pandas regardless of which engine you use for the rest.
- **Python 3.10+** with `pandas` and `numpy` — required this week for the anomaly-detection pipeline (Lecture 3, Exercise 3, Challenge 2, the mini-project):
  ```bash
  pip install pandas numpy
  ```

## Required reading (this week's core)

- **PostgreSQL — Window Functions Tutorial:** <https://www.postgresql.org/docs/current/tutorial-window.html>
  *Why: the rolling-baseline mechanism in Lecture 2 and Exercise 3 is built entirely on `OVER (... ROWS BETWEEN ... PRECEDING ...)`.*
- **PostgreSQL — Materialized Views:** <https://www.postgresql.org/docs/current/rules-materializedviews.html>
  *Why: Lecture 2, Section 4 — the pattern a production control tower uses to keep a rolling-baseline query fast at scale.*
- **pandas — `rolling()` window documentation:** <https://pandas.pydata.org/docs/reference/window.html>
  *Why: Exercise 3 and Challenge 2's entire pipeline is `.shift(1).rolling(14).mean()` / `.std()` — this page is the reference for every variant you'll need.*
- **MIT Center for Transportation & Logistics — Supply Chain Resilience research:** <https://ctl.mit.edu/research/supply-chain-resilience>
  *Why: the academic grounding behind Lecture 1's risk taxonomy and resilience-lever framework.*

## Reference (keep in tabs)

- **ISO 28000 — Security and resilience management systems for the supply chain (overview):** <https://www.iso.org/standard/79612.html>
  *Why: the industry-standard framing for supply chain risk management, if you want the formal standard behind this week's informal treatment.*
- **Apache Airflow — Concepts (DAGs, scheduling, retries):** <https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/overview.html>
  *Why: Lecture 2, Section 5 — what a production pipeline scheduler looks like once a `cron` job outgrows a single script.*
- **scikit-learn — Isolation Forest documentation:** <https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.IsolationForest.html>
  *Why: Lecture 3's pointer toward multivariate ML anomaly detection, for when a single-metric z-score isn't enough.*
- **Google Cloud — "What is an AI agent?":** <https://cloud.google.com/discover/what-are-ai-agents>
  *Why: a vendor-neutral framing of the agentic-automation boundary discussed in Lecture 3, Section 4.*
- **Council of Supply Chain Management Professionals — glossary of logistics terms:** <https://cscmp.org/CSCMP/Educate/SCM_Definitions_and_Glossary_of_Terms.aspx>
  *Why: industry-standard definitions any time this week's vocabulary feels unfamiliar.*

## Practice beyond the seed data

- **PostgreSQL Exercises** (general SQL aggregation and window-function practice): <https://pgexercises.com/>
- **NIST** — search "anomaly detection" for general statistical framing that extends beyond this week's operational-metrics use case: <https://www.nist.gov/>

## Glossary

| Term | Definition |
|------|------------|
| **Risk register** | A structured table of tracked risks, each scored on likelihood and impact, with a mitigation status. |
| **Single point of failure (SPOF)** | A node or lane whose loss stops or badly degrades a function the network has no other way to perform. |
| **Likelihood × impact matrix** | A 2-axis prioritization tool scoring risks on how often they occur and how bad one event is. |
| **Expected annual loss** | `likelihood (probability/year) × impact (cost/event)` — the dollar figure a resilience investment is measured against. |
| **Buffers** | A resilience lever: safety stock, capacity slack, or schedule time held to absorb a disruption. |
| **Redundancy** | A resilience lever: a qualified second supplier, carrier, or facility that removes a SPOF structurally. |
| **Flexibility** | A resilience lever: interchangeable capacity or modular design that lets the network reroute around a gap. |
| **Visibility** | A resilience lever: faster detection of a disruption, shrinking the time before a response begins. |
| **Control tower** | A continuously updated data layer, computed baselines/thresholds, and an alerting/escalation path, working together. |
| **Rolling baseline** | A "normal" range computed from a recent trailing window (e.g., the prior 14 days), recomputed every day. |
| **Materialized view** | A stored, schedule-refreshed query result — faster to read than recomputing the query every time. |
| **Z-score** | `(value - rolling mean) / rolling standard deviation` — a deviation measured in units of normal volatility. |
| **IQR (interquartile range) method** | An outlier-detection method using `Q3 - Q1` and a 1.5× fence, computed over a static dataset. |
| **Exception management** | Rules (persistence, severity tiers, auto-resolution) that decide which statistical flags actually reach a human. |
| **Alert fatigue** | The failure mode where too many low-value alerts cause a team to start ignoring the alert channel entirely. |
| **Agentic automation** | Software that can draft, or within guardrails execute, the next step after an anomaly is detected. |
| **OTIF (on-time-in-full)** | Share of orders shipped both complete and on schedule — this week's primary service-level metric. |

---

## Full `network_risk_register` seed data

The complete 16-row `INSERT` referenced in the [week README](./README.md).

```sql
INSERT INTO network_risk_register VALUES
(1, 'Andes Stitch Works', 'Supplier', 'Supplier', 'Sole qualified CMT line for the flagship Outerwear style; no second line trained on the pattern', 3, 4, TRUE, 35, 'Partial'),
(2, 'Alpine Weave Mills', 'Supplier', 'Supplier', 'Largest fabric mill by volume; 2 smaller mills qualified but slow to ramp', 2, 4, FALSE, 40, 'Partial'),
(3, 'Hai Phong CM Region', 'Facility', 'Geopolitical', 'Sole offshore contract-manufacturing region and import gateway; typhoon season closes the port 3-5 times/yr', 4, 5, TRUE, 60, 'None'),
(4, 'Pacific Rim Ocean Lines', 'Carrier', 'Transportation', 'Sole ocean carrier booked on the HPH-ATX import lane', 2, 3, TRUE, 100, 'Partial'),
(5, 'Austin East DC', 'Facility', 'Facility', 'Largest DC by volume; single ERCOT grid connection, no generator failover, winter-storm exposure', 3, 5, TRUE, 45, 'None'),
(6, 'Memphis DC', 'Facility', 'Facility', 'Tornado-corridor exposure; regional roof/power damage risk', 3, 3, FALSE, 30, 'Partial'),
(7, 'Reno DC', 'Facility', 'Facility', 'Wildfire-smoke exposure can force temporary dock closures', 2, 3, FALSE, 25, 'Partial'),
(8, 'Order Management System', 'System', 'Cyber', 'Single ERP/OMS instance; no hot failover; ransomware or outage halts order release network-wide', 2, 5, TRUE, 100, 'None'),
(9, 'Longhaul Truckload', 'Carrier', 'Transportation', 'Dominant FTL carrier on DC-to-DC replenishment lanes', 2, 3, FALSE, 38, 'Partial'),
(10, 'ButtonWorks Supply', 'Supplier', 'Supplier', 'Sole qualified hardware/zipper source for the Outerwear line', 2, 3, TRUE, 55, 'Partial'),
(11, 'Top Wholesale Account', 'Market', 'Demand', 'Single chain account concentrates a large share of wholesale revenue', 2, 4, TRUE, 22, 'None'),
(12, 'Regional Rail Interchange', 'Carrier', 'Transportation', 'Single Class-I interchange yard carries all Intermodal DC-to-DC volume', 2, 3, TRUE, 28, 'Partial'),
(13, 'Vietnam Trade Policy', 'Market', 'Geopolitical', 'Tariff or duty-rate change on CM imports from the sole sourcing country', 3, 4, TRUE, 60, 'None'),
(14, 'Northstar Apparel Mfg', 'Supplier', 'Supplier', 'Sole CMT source for the Accessories line; smaller, easier to dual-source', 2, 2, TRUE, 100, 'Mitigated'),
(15, 'SkyBridge Air Cargo', 'Carrier', 'Transportation', 'Sole qualified air-freight carrier for HPH-ATX expedites', 2, 2, TRUE, 100, 'Mitigated'),
(16, 'Austin East Regional Labor Market', 'Facility', 'Facility', 'Tight seasonal labor market around peak; understaffing risk, not a single event but a recurring capacity risk', 3, 3, FALSE, NULL, 'Partial');
```

Sanity check after loading: `SELECT COUNT(*) FROM network_risk_register;` should print `16`.

---

## Full `daily_ops` seed data

The complete 180-row `INSERT` referenced in the [week README](./README.md) — 60 days (2026-04-01 to 2026-05-30), 3 DCs. The Andes Stitch Works disruption (fire 2026-04-15, visible Austin East impact 2026-04-27 through 2026-05-20) is baked into these rows.

```sql
INSERT INTO daily_ops VALUES
(1,'2026-04-01','Austin East',156,1445,95.5,96.7,2.0,FALSE),
(2,'2026-04-01','Memphis DC',120,797,94.7,97.0,2.2,FALSE),
(3,'2026-04-01','Reno DC',104,955,92.8,94.6,2.4,FALSE),
(4,'2026-04-02','Austin East',149,942,95.5,96.9,2.2,FALSE),
(5,'2026-04-02','Memphis DC',126,1005,95.2,95.1,2.3,FALSE),
(6,'2026-04-02','Reno DC',108,702,94.6,94.0,2.6,FALSE),
(7,'2026-04-03','Austin East',170,1093,95.5,97.1,2.0,FALSE),
(8,'2026-04-03','Memphis DC',138,1078,95.3,95.3,2.2,FALSE),
(9,'2026-04-03','Reno DC',106,987,94.7,94.1,2.4,FALSE),
(10,'2026-04-04','Austin East',162,1281,96.8,97.1,2.2,FALSE),
(11,'2026-04-04','Memphis DC',119,1063,93.8,96.9,2.2,FALSE),
(12,'2026-04-04','Reno DC',100,886,94.1,94.5,2.2,FALSE),
(13,'2026-04-05','Austin East',153,1102,95.1,96.4,2.1,FALSE),
(14,'2026-04-05','Memphis DC',132,1065,95.7,95.2,2.2,FALSE),
(15,'2026-04-05','Reno DC',119,968,93.8,94.2,2.4,FALSE),
(16,'2026-04-06','Austin East',153,1262,95.4,97.7,2.3,FALSE),
(17,'2026-04-06','Memphis DC',137,1195,95.1,95.9,2.2,FALSE),
(18,'2026-04-06','Reno DC',107,948,95.0,94.5,2.3,FALSE),
(19,'2026-04-07','Austin East',168,1045,96.7,97.4,2.1,FALSE),
(20,'2026-04-07','Memphis DC',136,1232,95.9,96.7,2.4,FALSE),
(21,'2026-04-07','Reno DC',100,643,92.9,95.3,2.5,FALSE),
(22,'2026-04-08','Austin East',162,1153,95.5,96.9,2.1,FALSE),
(23,'2026-04-08','Memphis DC',124,1006,94.9,95.5,2.4,FALSE),
(24,'2026-04-08','Reno DC',105,978,94.1,94.1,2.4,FALSE),
(25,'2026-04-09','Austin East',158,1242,96.7,97.5,2.0,FALSE),
(26,'2026-04-09','Memphis DC',123,1116,96.2,96.1,2.5,FALSE),
(27,'2026-04-09','Reno DC',117,971,93.0,95.3,2.3,FALSE),
(28,'2026-04-10','Austin East',163,1330,94.8,96.1,2.3,FALSE),
(29,'2026-04-10','Memphis DC',126,937,95.7,96.9,2.3,FALSE),
(30,'2026-04-10','Reno DC',116,825,95.0,94.5,2.3,FALSE),
(31,'2026-04-11','Austin East',168,1181,96.6,97.9,2.0,FALSE),
(32,'2026-04-11','Memphis DC',125,758,94.6,95.6,2.3,FALSE),
(33,'2026-04-11','Reno DC',116,958,94.3,95.6,2.5,FALSE),
(34,'2026-04-12','Austin East',159,1141,95.2,96.5,2.0,FALSE),
(35,'2026-04-12','Memphis DC',134,822,94.5,95.3,2.4,FALSE),
(36,'2026-04-12','Reno DC',114,827,94.3,95.6,2.3,FALSE),
(37,'2026-04-13','Austin East',148,1372,96.6,97.9,2.1,FALSE),
(38,'2026-04-13','Memphis DC',137,1252,94.2,95.4,2.1,FALSE),
(39,'2026-04-13','Reno DC',101,914,92.9,95.1,2.4,FALSE),
(40,'2026-04-14','Austin East',170,1255,96.4,97.3,2.2,FALSE),
(41,'2026-04-14','Memphis DC',129,935,94.7,96.8,2.1,FALSE),
(42,'2026-04-14','Reno DC',113,1029,95.0,94.8,2.6,FALSE),
(43,'2026-04-15','Austin East',149,1368,97.1,96.8,2.1,FALSE),
(44,'2026-04-15','Memphis DC',130,837,95.9,96.9,2.1,FALSE),
(45,'2026-04-15','Reno DC',104,847,92.9,94.1,2.3,FALSE),
(46,'2026-04-16','Austin East',160,1494,95.3,96.4,2.3,FALSE),
(47,'2026-04-16','Memphis DC',142,1106,94.5,95.8,2.5,FALSE),
(48,'2026-04-16','Reno DC',100,927,93.8,94.4,2.6,FALSE),
(49,'2026-04-17','Austin East',166,1094,96.9,97.0,2.2,FALSE),
(50,'2026-04-17','Memphis DC',127,1178,95.4,96.6,2.3,FALSE),
(51,'2026-04-17','Reno DC',109,953,93.1,95.1,2.3,FALSE),
(52,'2026-04-18','Austin East',160,1124,93.6,96.6,2.0,TRUE),
(53,'2026-04-18','Memphis DC',140,1288,95.1,95.5,2.2,FALSE),
(54,'2026-04-18','Reno DC',120,789,93.0,94.9,2.4,FALSE),
(55,'2026-04-19','Austin East',156,1345,96.9,96.6,2.2,FALSE),
(56,'2026-04-19','Memphis DC',125,923,94.3,97.0,2.2,FALSE),
(57,'2026-04-19','Reno DC',103,965,94.1,95.1,2.5,FALSE),
(58,'2026-04-20','Austin East',160,1150,95.9,96.8,2.3,FALSE),
(59,'2026-04-20','Memphis DC',124,813,94.5,95.6,2.5,FALSE),
(60,'2026-04-20','Reno DC',119,1027,93.8,94.2,2.5,FALSE),
(61,'2026-04-21','Austin East',161,1221,95.3,97.0,2.0,FALSE),
(62,'2026-04-21','Memphis DC',130,1109,96.2,96.0,2.5,FALSE),
(63,'2026-04-21','Reno DC',106,922,93.5,94.8,2.5,FALSE),
(64,'2026-04-22','Austin East',165,1413,96.3,97.7,1.9,FALSE),
(65,'2026-04-22','Memphis DC',124,883,94.6,96.7,2.1,FALSE),
(66,'2026-04-22','Reno DC',118,1080,94.9,94.6,2.4,FALSE),
(67,'2026-04-23','Austin East',153,1218,95.3,97.2,2.1,FALSE),
(68,'2026-04-23','Memphis DC',122,1019,94.4,95.3,2.3,FALSE),
(69,'2026-04-23','Reno DC',120,879,93.1,95.2,2.4,FALSE),
(70,'2026-04-24','Austin East',152,1264,95.9,96.4,2.0,FALSE),
(71,'2026-04-24','Memphis DC',142,1065,95.7,95.5,2.4,FALSE),
(72,'2026-04-24','Reno DC',102,923,93.0,94.4,2.5,FALSE),
(73,'2026-04-25','Austin East',148,1070,96.3,96.3,2.2,FALSE),
(74,'2026-04-25','Memphis DC',122,1112,94.7,95.6,2.3,FALSE),
(75,'2026-04-25','Reno DC',120,863,92.8,94.0,2.4,FALSE),
(76,'2026-04-26','Austin East',152,1328,94.1,96.1,2.0,TRUE),
(77,'2026-04-26','Memphis DC',140,1233,93.8,96.2,2.4,FALSE),
(78,'2026-04-26','Reno DC',101,826,94.5,95.8,2.2,FALSE),
(79,'2026-04-27','Austin East',163,978,95.8,97.8,2.1,FALSE),
(80,'2026-04-27','Memphis DC',137,863,95.3,96.8,2.2,FALSE),
(81,'2026-04-27','Reno DC',109,779,94.3,95.6,2.5,FALSE),
(82,'2026-04-28','Austin East',150,1382,92.2,93.4,2.7,FALSE),
(83,'2026-04-28','Memphis DC',132,968,94.0,95.9,2.4,FALSE),
(84,'2026-04-28','Reno DC',119,769,92.8,94.8,2.4,FALSE),
(85,'2026-04-29','Austin East',148,1204,86.1,88.2,3.3,FALSE),
(86,'2026-04-29','Memphis DC',137,908,95.4,95.6,2.4,FALSE),
(87,'2026-04-29','Reno DC',105,640,94.0,95.9,2.6,FALSE),
(88,'2026-04-30','Austin East',163,1162,82.4,84.4,3.7,TRUE),
(89,'2026-04-30','Memphis DC',120,967,94.8,96.1,2.2,FALSE),
(90,'2026-04-30','Reno DC',103,661,92.8,95.6,2.5,FALSE),
(91,'2026-05-01','Austin East',151,1158,76.1,80.3,4.7,TRUE),
(92,'2026-05-01','Memphis DC',142,1081,95.4,96.6,2.2,FALSE),
(93,'2026-05-01','Reno DC',109,958,93.1,95.9,2.5,FALSE),
(94,'2026-05-02','Austin East',142,1137,72.6,75.2,4.9,TRUE),
(95,'2026-05-02','Memphis DC',119,1060,95.7,95.8,2.2,FALSE),
(96,'2026-05-02','Reno DC',120,983,93.5,94.2,2.5,FALSE),
(97,'2026-05-03','Austin East',149,1271,67.4,71.4,5.8,TRUE),
(98,'2026-05-03','Memphis DC',131,845,94.1,96.9,2.5,FALSE),
(99,'2026-05-03','Reno DC',104,756,94.0,94.5,2.5,FALSE),
(100,'2026-05-04','Austin East',138,1269,62.8,66.1,6.4,TRUE),
(101,'2026-05-04','Memphis DC',142,1267,95.1,96.1,2.4,FALSE),
(102,'2026-05-04','Reno DC',115,885,92.8,92.4,2.5,FALSE),
(103,'2026-05-05','Austin East',148,1208,61.8,68.0,6.5,TRUE),
(104,'2026-05-05','Memphis DC',123,1106,93.4,94.4,2.1,FALSE),
(105,'2026-05-05','Reno DC',101,868,95.1,94.4,2.5,FALSE),
(106,'2026-05-06','Austin East',141,1121,63.0,66.7,6.4,TRUE),
(107,'2026-05-06','Memphis DC',132,845,95.6,95.6,2.2,FALSE),
(108,'2026-05-06','Reno DC',110,889,95.0,95.7,2.6,FALSE),
(109,'2026-05-07','Austin East',146,1134,61.8,67.5,6.5,TRUE),
(110,'2026-05-07','Memphis DC',136,1131,94.2,95.1,2.3,FALSE),
(111,'2026-05-07','Reno DC',99,923,93.7,95.1,2.3,FALSE),
(112,'2026-05-08','Austin East',133,917,62.0,67.0,6.4,TRUE),
(113,'2026-05-08','Memphis DC',120,797,95.6,95.6,2.4,FALSE),
(114,'2026-05-08','Reno DC',117,918,95.0,94.9,2.2,FALSE),
(115,'2026-05-09','Austin East',142,1029,62.1,66.1,6.2,TRUE),
(116,'2026-05-09','Memphis DC',140,1307,95.4,96.7,2.5,FALSE),
(117,'2026-05-09','Reno DC',119,809,91.4,93.6,2.6,FALSE),
(118,'2026-05-10','Austin East',150,916,61.0,67.4,6.4,TRUE),
(119,'2026-05-10','Memphis DC',140,1305,94.5,97.0,2.3,FALSE),
(120,'2026-05-10','Reno DC',117,888,93.3,94.6,2.3,FALSE),
(121,'2026-05-11','Austin East',153,1388,65.0,69.5,6.0,TRUE),
(122,'2026-05-11','Memphis DC',138,1137,95.9,95.1,2.4,FALSE),
(123,'2026-05-11','Reno DC',110,833,94.3,95.3,2.5,FALSE),
(124,'2026-05-12','Austin East',158,1187,68.2,72.6,5.5,TRUE),
(125,'2026-05-12','Memphis DC',119,1009,94.4,96.2,2.4,FALSE),
(126,'2026-05-12','Reno DC',107,677,95.0,95.6,2.3,FALSE),
(127,'2026-05-13','Austin East',141,1056,72.7,76.2,5.1,FALSE),
(128,'2026-05-13','Memphis DC',136,838,96.1,96.7,2.4,FALSE),
(129,'2026-05-13','Reno DC',120,751,94.7,95.6,2.3,FALSE),
(130,'2026-05-14','Austin East',138,1199,75.2,78.4,4.7,TRUE),
(131,'2026-05-14','Memphis DC',126,937,95.4,95.9,2.4,FALSE),
(132,'2026-05-14','Reno DC',103,916,93.4,96.0,2.6,FALSE),
(133,'2026-05-15','Austin East',146,893,79.4,82.8,4.2,TRUE),
(134,'2026-05-15','Memphis DC',141,1284,95.3,95.8,2.4,FALSE),
(135,'2026-05-15','Reno DC',109,997,95.2,95.2,2.3,FALSE),
(136,'2026-05-16','Austin East',145,1260,82.6,85.0,3.8,FALSE),
(137,'2026-05-16','Memphis DC',122,953,94.9,96.8,2.2,FALSE),
(138,'2026-05-16','Reno DC',112,763,94.3,95.5,2.6,FALSE),
(139,'2026-05-17','Austin East',146,1096,84.7,87.1,3.3,FALSE),
(140,'2026-05-17','Memphis DC',126,1002,95.2,95.1,2.2,FALSE),
(141,'2026-05-17','Reno DC',111,924,94.4,94.7,2.4,FALSE),
(142,'2026-05-18','Austin East',153,977,89.9,90.9,3.0,TRUE),
(143,'2026-05-18','Memphis DC',142,1333,95.7,97.0,2.2,FALSE),
(144,'2026-05-18','Reno DC',118,725,94.5,96.0,2.5,FALSE),
(145,'2026-05-19','Austin East',152,1074,93.5,93.3,2.3,FALSE),
(146,'2026-05-19','Memphis DC',128,791,94.8,96.0,2.3,FALSE),
(147,'2026-05-19','Reno DC',120,801,94.1,94.7,2.5,FALSE),
(148,'2026-05-20','Austin East',158,1287,96.3,98.0,1.9,FALSE),
(149,'2026-05-20','Memphis DC',139,1057,95.1,95.5,2.2,FALSE),
(150,'2026-05-20','Reno DC',116,1028,94.3,95.3,2.3,FALSE),
(151,'2026-05-21','Austin East',170,1510,95.0,96.8,2.0,FALSE),
(152,'2026-05-21','Memphis DC',138,979,95.1,95.5,2.4,FALSE),
(153,'2026-05-21','Reno DC',115,914,94.6,94.9,2.5,FALSE),
(154,'2026-05-22','Austin East',155,930,96.0,97.9,1.9,FALSE),
(155,'2026-05-22','Memphis DC',142,1331,95.4,95.0,2.2,FALSE),
(156,'2026-05-22','Reno DC',112,690,95.0,94.8,2.3,FALSE),
(157,'2026-05-23','Austin East',163,1121,96.2,97.4,2.2,FALSE),
(158,'2026-05-23','Memphis DC',121,858,96.1,95.1,2.2,FALSE),
(159,'2026-05-23','Reno DC',113,1018,95.0,94.1,2.4,FALSE),
(160,'2026-05-24','Austin East',171,1229,96.2,96.6,2.2,FALSE),
(161,'2026-05-24','Memphis DC',131,1070,95.7,95.0,2.3,FALSE),
(162,'2026-05-24','Reno DC',108,682,95.2,94.7,2.2,FALSE),
(163,'2026-05-25','Austin East',165,1240,95.4,96.5,1.9,FALSE),
(164,'2026-05-25','Memphis DC',121,974,93.9,96.5,2.3,FALSE),
(165,'2026-05-25','Reno DC',101,613,92.8,95.1,2.4,FALSE),
(166,'2026-05-26','Austin East',153,918,95.9,96.7,2.2,FALSE),
(167,'2026-05-26','Memphis DC',122,974,94.9,96.4,2.3,FALSE),
(168,'2026-05-26','Reno DC',107,976,94.6,94.8,2.6,FALSE),
(169,'2026-05-27','Austin East',167,1184,95.9,96.2,2.0,FALSE),
(170,'2026-05-27','Memphis DC',142,860,95.6,96.9,2.2,FALSE),
(171,'2026-05-27','Reno DC',99,847,93.4,95.9,2.4,FALSE),
(172,'2026-05-28','Austin East',161,977,95.9,96.9,1.9,FALSE),
(173,'2026-05-28','Memphis DC',128,1147,95.7,96.0,2.1,FALSE),
(174,'2026-05-28','Reno DC',122,965,94.7,94.1,2.5,FALSE),
(175,'2026-05-29','Austin East',159,1270,95.9,96.0,2.1,FALSE),
(176,'2026-05-29','Memphis DC',119,963,95.2,96.5,2.1,FALSE),
(177,'2026-05-29','Reno DC',112,744,93.9,94.3,2.3,FALSE),
(178,'2026-05-30','Austin East',158,1410,95.9,96.2,2.0,FALSE),
(179,'2026-05-30','Memphis DC',122,989,94.3,95.8,2.2,FALSE),
(180,'2026-05-30','Reno DC',104,687,92.8,95.4,2.3,FALSE);
```

Sanity check after loading: `SELECT COUNT(*) FROM daily_ops;` should print `180`.

---

*Broken link? Open an issue or PR.*
