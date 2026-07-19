# Week 8 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific question comes up. The full `pick_lines` seed data lives at the bottom of this page to keep the week [README](./README.md) short.

## Install first

- **PostgreSQL 16+** — the course's primary engine:
  <https://www.postgresql.org/download/> · macOS: [Postgres.app](https://postgresapp.com/) is the easiest. Linux: `sudo apt install postgresql` / `sudo dnf install postgresql-server`. Windows: the EDB installer.
- **SQLite 3.35+** — the zero-setup fallback; ships on macOS and most Linux already: <https://www.sqlite.org/download.html>. Check with `sqlite3 --version`.
- **Python 3.10+ with pandas and numpy:**
  ```bash
  pip install pandas numpy sqlalchemy psycopg2-binary
  ```
  No new libraries beyond earlier weeks — this week's math is arithmetic and `cumsum()`, not statistics.
- **A GUI (optional)** — [DBeaver](https://dbeaver.io/) (free, both engines) or [pgAdmin](https://www.pgadmin.org/) (Postgres).

## Required reading (this week's core)

- **Wikipedia — Pareto principle:** <https://en.wikipedia.org/wiki/Pareto_principle>
  *Why: the general 80/20 idea behind ABC analysis, with examples well beyond warehousing — useful for seeing that this week's method is one instance of a much more general pattern.*
- **PostgreSQL — Window Functions Tutorial:** <https://www.postgresql.org/docs/current/tutorial-window.html>
  *Why: the official walkthrough of the `OVER (ORDER BY ... ROWS BETWEEN ...)` syntax Lecture 2 builds the entire cumulative-share calculation on.*
- **pandas — `cumsum` documentation:** <https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.cumsum.html>
  *Why: the exact method signature for the pandas equivalent of this week's SQL running totals.*
- **NIST/SEMATECH e-Handbook — Pareto chart:** <https://www.itl.nist.gov/div898/handbook/pmc/section3/pmc32.htm>
  *Why: a rigorous, free reference on visualizing exactly the cumulative-percentage curve Lecture 2 computes in table form.*

## Reference (keep in tabs)

- **APICS/ASCM — CSCP body of knowledge (warehousing & DC operations section):** <https://www.ascm.org/learning-development/certifications-credentials/cscp/>
  *Why: the industry-standard certification body's treatment of slotting, picking methods, and DC labor metrics — worth skimming even outside certification prep.*
- **MHI (Material Handling Industry) — warehouse operations resources:** <https://www.mhi.org/>
  *Why: an industry association's practitioner-level material on picking technologies and slotting strategy, a useful contrast to this week's teaching simplifications.*
- **PostgreSQL — `CEIL`, `FLOOR`, and other math functions:** <https://www.postgresql.org/docs/current/functions-math.html>
  *Why: the exact signatures for `CEIL()` (used in Challenge 1's replenishment-trip calculation) and related rounding functions.*
- **NumPy — `numpy.ceil`:** <https://numpy.org/doc/stable/reference/generated/numpy.ceil.html>
  *Why: the pandas/numpy equivalent of SQL's `CEIL()`, used throughout the labor-sizing calculations in Lecture 3 and Challenge 2.*

## Practice beyond this week's catalog

- **MIT OpenCourseWare — Supply Chain Management (search current course listings):** <https://ocw.mit.edu/search/?q=supply%20chain>
  *Why: free graduate-level treatment of warehouse and distribution operations, if you want to go deeper than this course's applied approach.*
- **APICS/ASCM sample CSCP practice questions:** <https://www.ascm.org/>
  *Why: more multiple-choice reps on the exact vocabulary (ABC analysis, golden zone, wave picking, cycle time) this week's quiz draws on.*

## Deeper background (optional this week)

- **Ford W. Harris and the origins of scientific inventory/warehouse management** — search university library databases for early 20th-century industrial-engineering treatments of "storage location assignment problem" if you want to see how far back the mathematical version of "where should this go" actually dates.
  *Why: slotting-as-optimization is not a new idea — it's one of the oldest applied operations research problems, older than the computers now used to solve it at scale.*

---

## Full `pick_lines` seed data

The complete 225-row insert for the [week setup](./README.md#table-2--pick_lines). Load this instead of the representative slice shown in the README.

```sql
INSERT INTO pick_lines VALUES
(1,1,'2025-06-03',3,2),
(2,2,'2025-06-16',11,1),
(3,2,'2025-06-16',1,2),
(4,3,'2025-06-03',7,1),
(5,4,'2025-06-03',1,1),
(6,4,'2025-06-03',1,2),
(7,4,'2025-06-03',1,2),
(8,5,'2025-06-26',9,3),
(9,5,'2025-06-26',8,1),
(10,6,'2025-06-23',3,2),
(11,7,'2025-06-24',3,1),
(12,7,'2025-06-24',2,1),
(13,7,'2025-06-24',1,3),
(14,8,'2025-06-24',5,1),
(15,8,'2025-06-24',3,1),
(16,9,'2025-06-09',3,2),
(17,9,'2025-06-09',6,1),
(18,9,'2025-06-09',4,3),
(19,9,'2025-06-09',8,1),
(20,9,'2025-06-09',1,1),
(21,10,'2025-06-03',3,1),
(22,10,'2025-06-03',5,1),
(23,10,'2025-06-03',3,3),
(24,11,'2025-06-24',7,1),
(25,11,'2025-06-24',4,2),
(26,11,'2025-06-24',1,1),
(27,12,'2025-06-04',4,1),
(28,12,'2025-06-04',7,2),
(29,12,'2025-06-04',11,1),
(30,12,'2025-06-04',1,2),
(31,13,'2025-06-09',17,2),
(32,13,'2025-06-09',2,1),
(33,13,'2025-06-09',1,2),
(34,14,'2025-06-04',1,1),
(35,14,'2025-06-04',17,1),
(36,14,'2025-06-04',1,1),
(37,14,'2025-06-04',1,2),
(38,14,'2025-06-04',12,1),
(39,15,'2025-06-27',1,2),
(40,15,'2025-06-27',1,1),
(41,16,'2025-06-04',2,3),
(42,16,'2025-06-04',2,1),
(43,16,'2025-06-04',14,2),
(44,16,'2025-06-04',2,2),
(45,17,'2025-06-11',3,1),
(46,17,'2025-06-11',12,1),
(47,17,'2025-06-11',4,1),
(48,18,'2025-06-18',5,1),
(49,18,'2025-06-18',2,1),
(50,18,'2025-06-18',2,1),
(51,18,'2025-06-18',3,3),
(52,19,'2025-06-05',1,3),
(53,19,'2025-06-05',4,1),
(54,19,'2025-06-05',12,1),
(55,19,'2025-06-05',1,1),
(56,20,'2025-06-26',1,2),
(57,20,'2025-06-26',1,1),
(58,20,'2025-06-26',2,2),
(59,21,'2025-06-11',2,1),
(60,21,'2025-06-11',5,1),
(61,22,'2025-06-26',3,3),
(62,22,'2025-06-26',3,2),
(63,23,'2025-06-27',2,3),
(64,23,'2025-06-27',1,1),
(65,23,'2025-06-27',15,2),
(66,23,'2025-06-27',2,2),
(67,24,'2025-06-03',3,1),
(68,24,'2025-06-03',1,1),
(69,24,'2025-06-03',3,3),
(70,25,'2025-06-27',2,1),
(71,25,'2025-06-27',9,3),
(72,25,'2025-06-27',1,3),
(73,25,'2025-06-27',11,1),
(74,26,'2025-06-04',4,1),
(75,26,'2025-06-04',1,1),
(76,26,'2025-06-04',3,1),
(77,26,'2025-06-04',6,2),
(78,27,'2025-06-19',16,1),
(79,27,'2025-06-19',18,3),
(80,27,'2025-06-19',1,1),
(81,27,'2025-06-19',16,3),
(82,28,'2025-06-26',8,2),
(83,29,'2025-06-26',9,3),
(84,29,'2025-06-26',1,2),
(85,30,'2025-06-24',7,3),
(86,31,'2025-06-16',4,1),
(87,31,'2025-06-16',2,1),
(88,31,'2025-06-16',13,1),
(89,31,'2025-06-16',10,3),
(90,32,'2025-06-12',6,2),
(91,32,'2025-06-12',4,1),
(92,32,'2025-06-12',1,1),
(93,33,'2025-06-10',4,3),
(94,33,'2025-06-10',13,1),
(95,34,'2025-06-16',20,3),
(96,34,'2025-06-16',7,1),
(97,34,'2025-06-16',2,1),
(98,35,'2025-06-11',3,1),
(99,35,'2025-06-11',19,3),
(100,35,'2025-06-11',1,1),
(101,36,'2025-06-12',1,1),
(102,36,'2025-06-12',3,1),
(103,36,'2025-06-12',1,3),
(104,37,'2025-06-18',6,3),
(105,37,'2025-06-18',10,3),
(106,37,'2025-06-18',3,1),
(107,37,'2025-06-18',4,1),
(108,37,'2025-06-18',15,1),
(109,38,'2025-06-06',3,2),
(110,38,'2025-06-06',7,1),
(111,38,'2025-06-06',6,1),
(112,38,'2025-06-06',1,1),
(113,39,'2025-06-13',12,1),
(114,39,'2025-06-13',4,3),
(115,40,'2025-06-20',2,1),
(116,40,'2025-06-20',3,3),
(117,40,'2025-06-20',12,1),
(118,40,'2025-06-20',2,1),
(119,41,'2025-06-16',4,2),
(120,41,'2025-06-16',4,2),
(121,42,'2025-06-04',17,1),
(122,42,'2025-06-04',8,2),
(123,42,'2025-06-04',3,3),
(124,43,'2025-06-02',2,2),
(125,43,'2025-06-02',3,1),
(126,43,'2025-06-02',11,1),
(127,43,'2025-06-02',1,1),
(128,44,'2025-06-20',1,1),
(129,44,'2025-06-20',4,1),
(130,44,'2025-06-20',3,2),
(131,45,'2025-06-27',2,1),
(132,45,'2025-06-27',14,1),
(133,46,'2025-06-26',6,1),
(134,46,'2025-06-26',1,2),
(135,46,'2025-06-26',2,1),
(136,46,'2025-06-26',11,1),
(137,47,'2025-06-05',2,2),
(138,47,'2025-06-05',5,3),
(139,47,'2025-06-05',3,1),
(140,47,'2025-06-05',5,1),
(141,47,'2025-06-05',18,2),
(142,48,'2025-06-04',4,2),
(143,48,'2025-06-04',1,1),
(144,49,'2025-06-25',1,1),
(145,50,'2025-06-10',2,2),
(146,50,'2025-06-10',1,1),
(147,50,'2025-06-10',2,1),
(148,50,'2025-06-10',2,1),
(149,50,'2025-06-10',1,2),
(150,51,'2025-06-24',2,1),
(151,52,'2025-06-12',3,1),
(152,52,'2025-06-12',9,2),
(153,53,'2025-06-06',10,3),
(154,53,'2025-06-06',1,2),
(155,53,'2025-06-06',13,2),
(156,54,'2025-06-17',7,2),
(157,54,'2025-06-17',2,2),
(158,55,'2025-06-04',2,2),
(159,55,'2025-06-04',8,2),
(160,55,'2025-06-04',1,1),
(161,55,'2025-06-04',1,1),
(162,56,'2025-06-11',5,1),
(163,56,'2025-06-11',2,1),
(164,56,'2025-06-11',6,2),
(165,56,'2025-06-11',6,1),
(166,57,'2025-06-17',2,2),
(167,57,'2025-06-17',4,1),
(168,57,'2025-06-17',1,2),
(169,57,'2025-06-17',3,3),
(170,58,'2025-06-13',1,1),
(171,58,'2025-06-13',4,2),
(172,58,'2025-06-13',3,1),
(173,59,'2025-06-09',1,2),
(174,59,'2025-06-09',2,1),
(175,59,'2025-06-09',13,2),
(176,59,'2025-06-09',2,1),
(177,60,'2025-06-20',2,1),
(178,61,'2025-06-25',1,1),
(179,61,'2025-06-25',3,2),
(180,62,'2025-06-13',14,1),
(181,62,'2025-06-13',16,2),
(182,62,'2025-06-13',2,1),
(183,62,'2025-06-13',14,1),
(184,63,'2025-06-27',1,1),
(185,64,'2025-06-24',10,2),
(186,64,'2025-06-24',2,1),
(187,64,'2025-06-24',15,2),
(188,64,'2025-06-24',8,2),
(189,65,'2025-06-02',1,1),
(190,65,'2025-06-02',1,2),
(191,65,'2025-06-02',2,2),
(192,66,'2025-06-25',5,1),
(193,66,'2025-06-25',3,1),
(194,66,'2025-06-25',2,2),
(195,66,'2025-06-25',3,3),
(196,67,'2025-06-13',15,1),
(197,67,'2025-06-13',9,1),
(198,67,'2025-06-13',7,1),
(199,68,'2025-06-05',9,1),
(200,68,'2025-06-05',1,1),
(201,68,'2025-06-05',4,3),
(202,68,'2025-06-05',19,2),
(203,69,'2025-06-06',5,2),
(204,69,'2025-06-06',4,2),
(205,69,'2025-06-06',2,1),
(206,69,'2025-06-06',2,1),
(207,69,'2025-06-06',1,1),
(208,70,'2025-06-12',1,1),
(209,70,'2025-06-12',1,2),
(210,70,'2025-06-12',3,1),
(211,70,'2025-06-12',5,2),
(212,71,'2025-06-05',1,1),
(213,71,'2025-06-05',4,3),
(214,72,'2025-06-05',18,3),
(215,72,'2025-06-05',8,2),
(216,72,'2025-06-05',1,1),
(217,73,'2025-06-25',10,2),
(218,73,'2025-06-25',1,1),
(219,73,'2025-06-25',2,1),
(220,74,'2025-06-06',2,2),
(221,74,'2025-06-06',2,2),
(222,74,'2025-06-06',10,2),
(223,75,'2025-06-12',1,2),
(224,76,'2025-06-13',1,1),
(225,76,'2025-06-13',6,1);
```

Sanity checks after loading:

```sql
SELECT COUNT(*) FROM pick_lines;                 -- 225
SELECT COUNT(DISTINCT order_id) FROM pick_lines;  -- 76
SELECT MIN(order_date), MAX(order_date) FROM pick_lines;  -- 2025-06-02 .. 2025-06-27
SELECT sku_id, COUNT(*) FROM pick_lines GROUP BY sku_id ORDER BY 2 DESC LIMIT 1;  -- sku_id 1, count 55
```
