# Week 7 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific question comes up. The full `shipments` seed data (all 124 rows, referenced from the [week README](./README.md)) is at the bottom of this page.

## Install first

- **PostgreSQL 16+** — the course's primary engine: <https://www.postgresql.org/download/> · macOS: [Postgres.app](https://postgresapp.com/). Linux: `sudo apt install postgresql` / `sudo dnf install postgresql-server`. Windows: the EDB installer.
- **SQLite 3.35+** — the zero-setup fallback, ships on macOS and most Linux already: <https://www.sqlite.org/download.html>. Check with `sqlite3 --version`.
- **Python 3.10+** with `pandas` and `numpy` — required this week for the routing exercises (Lecture 2, Exercise 2, Challenge 1):
  ```bash
  pip install pandas numpy
  ```
- **(Optional) matplotlib**, if you want to try Exercise 2's stretch goal of plotting the delivery network: `pip install matplotlib`.

## Required reading (this week's core)

- **PostgreSQL — Aggregate Functions:** <https://www.postgresql.org/docs/current/functions-aggregate.html>
  *Why: every lane-cost and carrier-scorecard query this week leans on `SUM`, `AVG`, and `COUNT` inside `GROUP BY`.*
- **PostgreSQL — The `HAVING` clause (in "The SQL Language" tutorial):** <https://www.postgresql.org/docs/current/tutorial-agg.html>
  *Why: the low-confidence-carrier filter in Lecture 3 and Exercise 3 depends on `HAVING COUNT(*) >= n`.*
- **Google OR-Tools — Vehicle Routing Problem guide:** <https://developers.google.com/optimization/routing/vrp>
  *Why: the production-grade version of everything Lecture 2 builds by hand — read it after Challenge 1 to see how a real solver formalizes the same ideas.*
- **Council of Supply Chain Management Professionals — glossary of logistics terms:** <https://cscmp.org/CSCMP/Educate/SCM_Definitions_and_Glossary_of_Terms.aspx>
  *Why: the industry-standard definitions for freight class, LTL, FTL, drayage, and more — useful any time this week's vocabulary feels unfamiliar.*

## Reference (keep in tabs)

- **U.S. DOT Bureau of Transportation Statistics — Freight Facts and Figures:** <https://www.bts.gov/product/freight-facts-and-figures>
  *Why: real U.S. freight volume and mode-share data, useful context for how the six modes actually compare at national scale.*
- **FMCSA — trucking regulations and weight limits:** <https://www.fmcsa.dot.gov/>
  *Why: the real legal gross-weight limits (~80,000 lb gross, ~45,000 lb typical dry-van payload) behind this week's FTL numbers.*
- **pandas documentation — `groupby`:** <https://pandas.pydata.org/docs/user_guide/groupby.html>
  *Why: Exercise 2 and Challenge 1 lean on Python loops more than pandas `groupby`, but you'll want this the moment you extend the routing code to summarize routes.*
- **Clarke, G. & Wright, J.W. (1964), "Scheduling of Vehicles from a Central Depot to a Number of Delivery Points"** — the original savings-algorithm paper, *Operations Research* 12(4), pp. 568-581. Available through most university library databases (JSTOR/INFORMS).
  *Why: the primary source behind Challenge 1 — reading the original 5-page paper after you've implemented it yourself is one of the best ways to confirm you actually understand it.*

## Practice beyond the seed data

- **Google OR-Tools examples (Python, free, open source):** <https://developers.google.com/optimization/routing/vrp> includes runnable CVRP and VRPTW code you can compare against your own Challenge 1 implementation.
- **PostgreSQL Exercises** (general SQL aggregation practice, not logistics-specific): <https://pgexercises.com/>

## Glossary

| Term | Definition |
|------|------------|
| **Parcel** | Small-package shipping (1-70 lb), priced by actual or dimensional weight and zone. |
| **LTL (Less-Than-Truckload)** | Shipments sharing trailer space with other freight, priced by weight class and distance. |
| **FTL (Full Truckload)** | One shipment fills a dedicated trailer, priced per mile or per load, not per pound. |
| **Intermodal** | Freight moved by truck-rail-truck combination; cheaper than FTL over long distances, slower. |
| **Freight class (NMFC)** | An LTL classification (1-500) based on density, handling difficulty, and liability. |
| **Weight break** | A published shipment-weight threshold where the per-cwt LTL rate drops. |
| **DIM weight (dimensional weight)** | Billed weight based on a package's volume, used when it exceeds actual weight. |
| **Cost per unit shipped** | Freight cost divided by units shipped — the metric that makes modes comparable regardless of shipment size. |
| **VRP (Vehicle Routing Problem)** | Find least-cost routes, from a depot, visiting a set of customer stops, respecting vehicle constraints. |
| **TSP (Traveling Salesman Problem)** | The single-vehicle, unlimited-capacity special case of the VRP. |
| **CVRP (Capacitated VRP)** | A VRP where each vehicle has a maximum load it cannot exceed. |
| **VRPTW** | A VRP where each stop must be visited within a specified time window. |
| **Nearest-neighbor heuristic** | A greedy routing algorithm: always travel to the closest unvisited stop next. |
| **Clarke-Wright savings algorithm** | A routing heuristic that merges single-stop routes based on the distance saved by pairing stops. |
| **On-time percentage** | Share of shipments where actual transit time was at or under the promised transit time. |
| **Freight spend** | Total dollars paid for transportation over a period; best tracked alongside cost-per-unit to separate volume growth from real cost changes. |

---

## Full `shipments` seed data

The complete 124-row `INSERT` referenced in the [week README](./README.md). Load this after creating the `shipments` table.

```sql
INSERT INTO shipments VALUES
(1,'2025-01-07','Austin East','Dallas, TX','AUS-DAL','Yellowline Freight','LTL',4787,195,1135.29,2,2,905),
(2,'2025-01-06','Austin East','Houston, TX','AUS-HOU','Longhaul Truckload','FTL',24104,165,1761.03,1,1,4895),
(3,'2025-01-06','Austin East','Denver, CO','AUS-DEN','Yellowline Freight','LTL',7672,925,2186.39,2,2,2045),
(4,'2025-01-09','Austin East','Denver, CO','AUS-DEN','RangeExpress','Parcel',31,925,45.89,2,1,13),
(5,'2025-01-10','Austin East','Phoenix, AZ','AUS-PHO','Basecamp Carriers','LTL',3123,900,914.35,2,2,795),
(6,'2025-01-10','Austin East','Atlanta, GA','AUS-ATL','Basecamp Carriers','LTL',8903,925,2529.11,2,2,2788),
(7,'2025-01-06','Austin East','Chicago, IL','AUS-CHI','SwiftPost','Parcel',32,1050,48.41,2,2,22),
(8,'2025-01-10','Austin East','Chicago, IL','AUS-CHI','Nimbus Parcel','Parcel',19,1050,31.7,2,4,8),
(9,'2025-01-07','Memphis DC','Chicago, IL','MEM-CHI','Longhaul Truckload','FTL',41627,530,3132.66,1,0,8633),
(10,'2025-01-07','Memphis DC','Atlanta, GA','MEM-ATL','Yellowline Freight','LTL',6551,385,1623.36,2,2,1375),
(11,'2025-01-07','Memphis DC','New York, NY','MEM-NEW','Basecamp Carriers','FTL',38993,1095,3379.85,2,2,11208),
(12,'2025-01-06','Memphis DC','Austin East','MEM-AUS','SwiftPost','Parcel',14,660,23.54,2,2,6),
(13,'2025-01-06','Reno DC','Los Angeles, CA','REN-LOS','Nimbus Parcel','Parcel',27,470,37.01,2,2,12),
(14,'2025-01-08','Reno DC','Seattle, WA','REN-SEA','SwiftPost','Parcel',7,730,15.57,2,1,4),
(15,'2025-01-10','Reno DC','Denver, CO','REN-DEN','SwiftPost','Parcel',21,965,33.7,2,2,11),
(16,'2025-01-10','Reno DC','Denver, CO','REN-DEN','Longhaul Truckload','FTL',10702,965,1103.77,2,1,2612),
(17,'2025-01-06','Reno DC','Austin East','REN-AUS','RangeExpress','Parcel',21,1495,37.23,2,4,13),
(18,'2025-01-07','Reno DC','Austin East','REN-AUS','Longhaul Truckload','FTL',28783,1495,2791.69,3,3,7179),
(19,'2025-01-16','Austin East','Dallas, TX','AUS-DAL','RangeExpress','Parcel',2,195,9.31,2,1,1),
(20,'2025-01-14','Austin East','Dallas, TX','AUS-DAL','Basecamp Carriers','FTL',41536,195,2856.3,1,1,9534),
(21,'2025-01-13','Austin East','Houston, TX','AUS-HOU','Nimbus Parcel','Parcel',39,165,46.45,2,4,27),
(22,'2025-01-17','Austin East','Denver, CO','AUS-DEN','CrossLTL','LTL',1423,925,446.58,2,2,469),
(23,'2025-01-14','Austin East','Phoenix, AZ','AUS-PHO','RangeExpress','Parcel',30,900,44.41,2,1,16),
(24,'2025-01-14','Austin East','Phoenix, AZ','AUS-PHO','Nimbus Parcel','Parcel',24,900,36.98,2,1,17),
(25,'2025-01-13','Austin East','Atlanta, GA','AUS-ATL','Yellowline Freight','LTL',1200,925,384.5,2,2,396),
(26,'2025-01-16','Austin East','Chicago, IL','AUS-CHI','Nimbus Parcel','Parcel',2,1050,9.85,2,2,1),
(27,'2025-01-16','Memphis DC','Chicago, IL','MEM-CHI','Longhaul Truckload','FTL',29083,530,2273.03,1,1,5519),
(28,'2025-01-14','Memphis DC','Atlanta, GA','MEM-ATL','Yellowline Freight','FTL',13748,385,1183.44,1,1,3112),
(29,'2025-01-17','Memphis DC','Atlanta, GA','MEM-ATL','CrossLTL','LTL',4546,385,1141.94,2,2,1444),
(30,'2025-01-14','Memphis DC','New York, NY','MEM-NEW','RangeExpress','Parcel',9,1095,18.98,2,2,4),
(31,'2025-01-14','Memphis DC','Austin East','MEM-AUS','Nimbus Parcel','Parcel',17,660,27.02,2,1,12),
(32,'2025-01-15','Reno DC','Los Angeles, CA','REN-LOS','Basecamp Carriers','FTL',21863,470,1752.78,1,1,5498),
(33,'2025-01-13','Reno DC','Los Angeles, CA','REN-LOS','CrossLTL','LTL',8528,470,2149.47,2,2,1518),
(34,'2025-01-16','Reno DC','Seattle, WA','REN-SEA','RailBridge Intermodal','Intermodal',37298,730,2213.92,3,3,11061),
(35,'2025-01-16','Reno DC','Denver, CO','REN-DEN','CrossLTL','LTL',3026,965,901.46,2,2,546),
(36,'2025-01-15','Reno DC','Denver, CO','REN-DEN','Basecamp Carriers','LTL',3190,965,947.59,2,1,738),
(37,'2025-01-17','Reno DC','Austin East','REN-AUS','RangeExpress','Parcel',38,1495,61.48,2,2,22),
(38,'2025-01-21','Austin East','Dallas, TX','AUS-DAL','SwiftPost','Parcel',24,195,31.61,2,1,16),
(39,'2025-01-20','Austin East','Houston, TX','AUS-HOU','SwiftPost','Parcel',23,165,30.38,2,1,18),
(40,'2025-01-24','Austin East','Houston, TX','AUS-HOU','Basecamp Carriers','FTL',21532,165,1602.99,1,1,6551),
(41,'2025-01-21','Austin East','Denver, CO','AUS-DEN','Nimbus Parcel','Parcel',3,925,11.02,2,1,2),
(42,'2025-01-20','Austin East','Phoenix, AZ','AUS-PHO','RangeExpress','Parcel',38,900,54.31,2,2,21),
(43,'2025-01-23','Austin East','Atlanta, GA','AUS-ATL','SwiftPost','Parcel',15,925,25.96,2,2,10),
(44,'2025-01-24','Austin East','Chicago, IL','AUS-CHI','RangeExpress','Parcel',22,1050,35.55,2,2,9),
(45,'2025-01-22','Memphis DC','Chicago, IL','MEM-CHI','Basecamp Carriers','LTL',2316,530,630.31,2,2,439),
(46,'2025-01-20','Memphis DC','Chicago, IL','MEM-CHI','Yellowline Freight','LTL',8334,530,2137.19,2,1,1935),
(47,'2025-01-24','Memphis DC','Atlanta, GA','MEM-ATL','Nimbus Parcel','Parcel',24,385,33.06,2,1,13),
(48,'2025-01-23','Memphis DC','New York, NY','MEM-NEW','RailBridge Intermodal','Intermodal',29547,1095,1983.83,3,3,6262),
(49,'2025-01-20','Memphis DC','Austin East','MEM-AUS','SwiftPost','Parcel',5,660,13.09,2,5,2),
(50,'2025-01-23','Reno DC','Los Angeles, CA','REN-LOS','SwiftPost','Parcel',28,470,38.11,2,2,12),
(51,'2025-01-21','Reno DC','Seattle, WA','REN-SEA','RangeExpress','Parcel',1,730,8.46,2,2,1),
(52,'2025-01-23','Reno DC','Denver, CO','REN-DEN','Longhaul Truckload','FTL',26155,965,2293.25,2,2,7440),
(53,'2025-01-21','Reno DC','Austin East','REN-AUS','Longhaul Truckload','Intermodal',27072,1495,1992.63,4,6,7467),
(54,'2025-01-31','Austin East','Dallas, TX','AUS-DAL','Nimbus Parcel','Parcel',25,195,32.63,2,2,17),
(55,'2025-01-31','Austin East','Dallas, TX','AUS-DAL','Yellowline Freight','LTL',7934,195,1848.5,2,2,1900),
(56,'2025-01-28','Austin East','Houston, TX','AUS-HOU','Yellowline Freight','FTL',38815,165,2664.92,1,1,9804),
(57,'2025-01-28','Austin East','Houston, TX','AUS-HOU','Basecamp Carriers','LTL',4375,165,1032.61,2,2,1205),
(58,'2025-01-30','Austin East','Denver, CO','AUS-DEN','RangeExpress','Parcel',5,925,13.51,2,1,2),
(59,'2025-01-28','Austin East','Denver, CO','AUS-DEN','Longhaul Truckload','FTL',19152,925,1739.33,2,2,3846),
(60,'2025-01-31','Austin East','Phoenix, AZ','AUS-PHO','Yellowline Freight','LTL',7020,900,1992.41,2,2,1744),
(61,'2025-01-27','Austin East','Phoenix, AZ','AUS-PHO','Yellowline Freight','LTL',4193,900,1210.35,2,2,736),
(62,'2025-01-27','Austin East','Atlanta, GA','AUS-ATL','Nimbus Parcel','Parcel',23,925,35.93,2,1,16),
(63,'2025-01-31','Austin East','Chicago, IL','AUS-CHI','SwiftPost','Parcel',33,1050,49.69,2,2,21),
(64,'2025-01-31','Memphis DC','Chicago, IL','MEM-CHI','Nimbus Parcel','Parcel',33,530,44.25,2,5,15),
(65,'2025-01-28','Memphis DC','Atlanta, GA','MEM-ATL','RangeExpress','Parcel',28,385,37.36,2,2,13),
(66,'2025-01-31','Memphis DC','Atlanta, GA','MEM-ATL','Nimbus Parcel','Parcel',38,385,48.1,2,1,18),
(67,'2025-01-30','Memphis DC','New York, NY','MEM-NEW','Longhaul Truckload','FTL',32124,1095,2833.78,2,2,6112),
(68,'2025-01-30','Memphis DC','New York, NY','MEM-NEW','RailBridge Intermodal','Intermodal',40369,1095,2546.35,3,2,12419),
(69,'2025-01-30','Memphis DC','Austin East','MEM-AUS','SwiftPost','Parcel',15,660,24.7,2,2,8),
(70,'2025-01-27','Memphis DC','Austin East','MEM-AUS','Longhaul Truckload','Intermodal',24542,660,1588.16,3,2,6825),
(71,'2025-01-29','Reno DC','Los Angeles, CA','REN-LOS','Basecamp Carriers','FTL',30548,470,2337.84,1,0,5114),
(72,'2025-01-31','Reno DC','Los Angeles, CA','REN-LOS','Nimbus Parcel','Parcel',26,470,35.91,2,2,11),
(73,'2025-01-29','Reno DC','Seattle, WA','REN-SEA','SwiftPost','Parcel',6,730,14.38,2,2,2),
(74,'2025-01-27','Reno DC','Seattle, WA','REN-SEA','Nimbus Parcel','Parcel',32,730,45.16,2,2,14),
(75,'2025-01-28','Reno DC','Denver, CO','REN-DEN','Basecamp Carriers','LTL',6667,965,1925.5,2,2,1206),
(76,'2025-01-31','Reno DC','Austin East','REN-AUS','Basecamp Carriers','FTL',41584,1495,3908.74,3,2,7460),
(77,'2025-01-27','Reno DC','Austin East','REN-AUS','Nimbus Parcel','Parcel',32,1495,52.93,2,1,17),
(78,'2025-02-07','Austin East','Dallas, TX','AUS-DAL','SwiftPost','Parcel',7,195,14.38,2,2,3),
(79,'2025-02-05','Austin East','Dallas, TX','AUS-DAL','Yellowline Freight','LTL',549,195,174.82,2,2,115),
(80,'2025-02-07','Austin East','Houston, TX','AUS-HOU','SwiftPost','Parcel',25,165,32.39,2,2,12),
(81,'2025-02-07','Austin East','Denver, CO','AUS-DEN','Yellowline Freight','FTL',27995,925,2413.14,2,2,5659),
(82,'2025-02-07','Austin East','Denver, CO','AUS-DEN','Basecamp Carriers','FTL',25542,925,2226.23,2,2,8129),
(83,'2025-02-05','Austin East','Phoenix, AZ','AUS-PHO','Nimbus Parcel','Parcel',8,900,17.18,2,2,4),
(84,'2025-02-05','Austin East','Phoenix, AZ','AUS-PHO','Longhaul Truckload','FTL',31505,900,2665.31,2,2,6092),
(85,'2025-02-06','Austin East','Atlanta, GA','AUS-ATL','RangeExpress','Parcel',11,925,20.98,2,1,7),
(86,'2025-02-03','Austin East','Chicago, IL','AUS-CHI','CrossLTL','LTL',554,1050,209.55,2,2,136),
(87,'2025-02-05','Austin East','Chicago, IL','AUS-CHI','CrossLTL','LTL',3612,1050,1088.06,2,3,685),
(88,'2025-02-04','Memphis DC','Chicago, IL','MEM-CHI','RangeExpress','Parcel',30,530,40.89,2,2,13),
(89,'2025-02-07','Memphis DC','Chicago, IL','MEM-CHI','CrossLTL','LTL',5442,530,1413.05,2,2,1288),
(90,'2025-02-06','Memphis DC','Atlanta, GA','MEM-ATL','Basecamp Carriers','FTL',19180,385,1540.4,1,1,4742),
(91,'2025-02-07','Memphis DC','Atlanta, GA','MEM-ATL','Yellowline Freight','LTL',1674,385,452.34,2,2,293),
(92,'2025-02-06','Memphis DC','New York, NY','MEM-NEW','Longhaul Truckload','FTL',13753,1095,1373.33,2,2,2897),
(93,'2025-02-06','Memphis DC','Austin East','MEM-AUS','Longhaul Truckload','FTL',29525,660,2377.83,1,0,5594),
(94,'2025-02-03','Memphis DC','Austin East','MEM-AUS','Yellowline Freight','FTL',28901,660,2333.5,1,1,5491),
(95,'2025-02-06','Reno DC','Los Angeles, CA','REN-LOS','Basecamp Carriers','LTL',6482,470,1645.87,2,1,1396),
(96,'2025-02-04','Reno DC','Seattle, WA','REN-SEA','Nimbus Parcel','Parcel',17,730,27.4,2,1,7),
(97,'2025-02-06','Reno DC','Denver, CO','REN-DEN','Basecamp Carriers','FTL',29384,965,2541.8,2,2,8096),
(98,'2025-02-07','Reno DC','Denver, CO','REN-DEN','SwiftPost','Parcel',11,965,21.12,2,1,8),
(99,'2025-02-06','Reno DC','Austin East','REN-AUS','Longhaul Truckload','Intermodal',39100,1495,2678.91,4,4,11408),
(100,'2025-02-12','Austin East','Dallas, TX','AUS-DAL','SwiftPost','Parcel',7,195,14.38,2,2,5),
(101,'2025-02-14','Austin East','Houston, TX','AUS-HOU','RangeExpress','Parcel',2,165,9.29,2,1,1),
(102,'2025-02-11','Austin East','Houston, TX','AUS-HOU','Longhaul Truckload','FTL',10717,165,938.49,1,1,2932),
(103,'2025-02-14','Austin East','Denver, CO','AUS-DEN','Nimbus Parcel','Parcel',18,925,29.7,2,2,8),
(104,'2025-02-10','Austin East','Phoenix, AZ','AUS-PHO','Basecamp Carriers','LTL',2491,900,739.51,2,2,555),
(105,'2025-02-11','Austin East','Atlanta, GA','AUS-ATL','RangeExpress','Parcel',6,925,14.75,2,2,3),
(106,'2025-02-12','Austin East','Chicago, IL','AUS-CHI','Basecamp Carriers','LTL',3058,1050,928.9,2,5,930),
(107,'2025-02-10','Memphis DC','Chicago, IL','MEM-CHI','Longhaul Truckload','Intermodal',31552,530,1861.77,3,3,9256),
(108,'2025-02-12','Memphis DC','Chicago, IL','MEM-CHI','RailBridge Intermodal','Intermodal',35693,530,2047.31,3,3,7037),
(109,'2025-02-10','Memphis DC','Atlanta, GA','MEM-ATL','Yellowline Freight','FTL',21831,385,1714.61,1,1,4696),
(110,'2025-02-11','Memphis DC','New York, NY','MEM-NEW','Nimbus Parcel','Parcel',29,1095,44.96,2,2,22),
(111,'2025-02-12','Memphis DC','Austin East','MEM-AUS','Longhaul Truckload','FTL',35067,660,2771.61,1,1,6869),
(112,'2025-02-13','Memphis DC','Austin East','MEM-AUS','Longhaul Truckload','FTL',36329,660,2861.28,1,0,8445),
(113,'2025-02-11','Reno DC','Los Angeles, CA','REN-LOS','SwiftPost','Parcel',1,470,8.38,2,2,1),
(114,'2025-02-10','Reno DC','Seattle, WA','REN-SEA','SwiftPost','Parcel',34,730,47.52,2,2,22),
(115,'2025-02-10','Reno DC','Seattle, WA','REN-SEA','RailBridge Intermodal','Intermodal',36876,730,2193.94,3,3,10297),
(116,'2025-02-14','Reno DC','Denver, CO','REN-DEN','Longhaul Truckload','Intermodal',16117,965,1259.15,3,5,4247),
(117,'2025-02-14','Reno DC','Denver, CO','REN-DEN','Yellowline Freight','FTL',24067,965,2132.53,2,2,7709),
(118,'2025-02-12','Reno DC','Austin East','REN-AUS','RailBridge Intermodal','Intermodal',19733,1495,1573.9,4,4,3430),
(119,'2025-01-07','Hai Phong, Vietnam','Austin East','HPH-ATX','Pacific Rim Ocean Lines','Ocean',24909,8100,2143.74,34,35,5535),
(120,'2025-01-07','Hai Phong, Vietnam','Austin East','HPH-ATX','SkyBridge Air Cargo','Air',1383,8100,5237.3,4,4,307),
(121,'2025-01-21','Hai Phong, Vietnam','Austin East','HPH-ATX','Pacific Rim Ocean Lines','Ocean',18075,8100,2049.43,34,39,4016),
(122,'2025-01-21','Hai Phong, Vietnam','Austin East','HPH-ATX','SkyBridge Air Cargo','Air',1698,8100,6213.8,4,3,377),
(123,'2025-02-04','Hai Phong, Vietnam','Austin East','HPH-ATX','Pacific Rim Ocean Lines','Ocean',21816,8100,2101.06,34,34,4848),
(124,'2025-02-04','Hai Phong, Vietnam','Austin East','HPH-ATX','SkyBridge Air Cargo','Air',1893,8100,6818.3,4,3,420);
```

Sanity check after loading: `SELECT COUNT(*) FROM shipments;` should print `124`.

---

*Broken link? Open an issue or PR.*
