# Week 7 — Quiz

Fifteen questions. Lectures closed. Aim for 12/15 before moving to Week 8. A mix of multiple-choice and short "what would you compute" questions — the answer key at the bottom explains the *why*, not just the letter.

---

**Q1.** Which transportation mode is typically priced **per mile or per load**, rather than by weight, once the shipment is booked?

- A) Parcel
- B) LTL
- C) FTL
- D) Air

---

**Q2.** A parcel box weighs 8 lb but measures 24"×18"×12" (5,184 in³). Using a DIM divisor of 139, the carrier will most likely bill based on:

- A) The actual weight, 8 lb, always
- B) The dimensional weight, ≈37 lb, because it's greater than actual weight
- C) Whichever weight is lower
- D) A flat rate regardless of size

---

**Q3.** Why can a 2,000-lb LTL shipment sometimes cost *less* in total than a 1,999-lb shipment on the same lane?

- A) LTL carriers round down for any shipment ending in "000"
- B) The 1,999-lb shipment is billed at a lower per-cwt rate break than the 2,000-lb shipment
- C) The 2,000-lb shipment crosses into a lower per-cwt rate break, and the lower rate applied to more weight nets out cheaper
- D) It never happens; heavier always costs more

---

**Q4.** "Cost per unit shipped" is a more useful comparison across modes than "average cost per shipment" mainly because:

- A) It's always a smaller number
- B) It removes the effect of wildly different shipment sizes across modes, isolating true per-item cost
- C) Finance requires it by law
- D) Average cost per shipment can't be computed in SQL

---

**Q5.** In the vehicle routing problem, the **Capacitated VRP (CVRP)** differs from the plain **Traveling Salesman Problem (TSP)** because CVRP adds:

- A) Time windows at each stop
- B) Multiple depots
- C) A maximum load per vehicle, potentially requiring more than one vehicle
- D) Pickup and delivery on the same route

---

**Q6.** Crunch Gear's Austin East network has 12 stops totaling 409 cases of demand, and each truck holds 120 cases. What is the **minimum possible** number of trucks needed, and why?

- A) 3, because `409 / 150 = 2.7` rounds to 3
- B) 4, because `409 / 120 = 3.4`, and you can't run a fractional truck
- C) 12, one per stop
- D) 1, if the driver makes multiple trips in one day

---

**Q7.** Why does straight-line (Euclidean) distance make sense as a simplification for *learning* the nearest-neighbor and savings algorithms, but not for a production routing system?

- A) It's mathematically impossible to compute in Python
- B) Real roads aren't straight lines, so it under- or over-states actual drive distance; production systems use road-network or drive-time data
- C) Euclidean distance only works for exactly 12 stops
- D) It's illegal to use in commercial routing software

---

**Q8.** The nearest-neighbor heuristic's main weakness is that it:

- A) Cannot handle more than 10 stops
- B) Always produces the mathematically optimal route
- C) Is greedy — it picks the closest next stop with no memory of where that choice leaves the route later, often producing an expensive final leg
- D) Requires a distance matrix that must be symmetric

---

**Q9.** The Clarke-Wright savings algorithm computes `savings(i, j) = dist(0, i) + dist(0, j) - dist(i, j)`. This quantity represents:

- A) The total distance of visiting both stops separately
- B) The miles saved by visiting `i` and `j` back-to-back on one truck instead of two separate round trips from the depot
- C) The extra distance caused by combining two routes
- D) The shortest possible route through all stops

---

**Q10.** In the Clarke-Wright algorithm, two routes can only be merged at a proposed pair `(i, j)` if:

- A) `i` and `j` have the highest total demand of any pair
- B) `i` and `j` are both endpoints of their (different) routes, and the merged load doesn't exceed capacity
- C) `i` and `j` are the two closest stops in the entire network
- D) The routes have never been merged before

---

**Q11.** On this week's 12-stop network, the unconstrained (single, infinite-capacity truck) nearest-neighbor route runs **243.8 miles**, while the capacity-constrained (120-case truck, 4 vehicles) version runs **345.7 miles**. The capacitated version is longer mainly because:

- A) The capacitated algorithm is buggy
- B) Splitting one big loop into several capacity-limited loops means more separate trips back to the depot
- C) Capacity constraints always double the distance
- D) The stops moved further apart

---

**Q12.** A carrier shows 33% on-time performance, but that figure is based on just 3 shipments. The correct interpretation is:

- A) The carrier is definitively unreliable — cut them immediately
- B) The sample is too small to draw a confident conclusion; flag as "watch," don't act decisively yet
- C) 33% on a small sample rounds up to acceptable
- D) Small samples are always more accurate than large ones

---

**Q13.** In a `HAVING COUNT(*) >= 10` clause added to a carrier scorecard query, the purpose is to:

- A) Speed up the query
- B) Exclude carriers whose on-time percentage is based on too few shipments to be statistically meaningful
- C) Limit results to exactly 10 rows
- D) Convert the query from an aggregate to a non-aggregate query

---

**Q14.** Crunch Gear needs 2,000 lb delivered in 5 days. Intermodal is cheapest (~$0.065/lb) but has a variable 3-7 day transit; LTL costs more (~$0.27/lb) but reliably delivers in 2 days; Air costs far more (~$3.67/lb) and delivers in 4 days. Given a hard 5-day deadline with meaningful cost if missed, the best call is usually:

- A) Air, because it's guaranteed fastest
- B) Intermodal, because it's cheapest
- C) LTL, because it's the cheapest option that reliably clears the actual deadline
- D) Whichever mode the carrier recommends

---

**Q15.** "Freight cost per unit" is generally more useful to track quarter-over-quarter than "total freight spend" because:

- A) Total freight spend is illegal to report to finance
- B) Freight cost per unit adjusts for shipment volume, so a rising figure signals a real efficiency problem rather than just more shipping activity
- C) Total freight spend can never be computed with SQL
- D) They are always equal, so it doesn't matter which one you track

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **C** — FTL is priced per truck/mile; weight up to the trailer's legal limit rides free once you've paid for the truck.
2. **B** — `5184 / 139 ≈ 37.3`, dimensional weight, which exceeds the 8 lb actual weight, so the carrier bills the greater of the two.
3. **C** — crossing a weight break lowers the per-cwt rate applied to the *entire* shipment, which can more than offset the extra 1 lb.
4. **B** — cost per unit normalizes for shipment size, letting you compare a Parcel box to an FTL trailer on the same footing.
5. **C** — CVRP adds a maximum vehicle capacity, which the plain TSP does not have.
6. **B** — `409 / 120 = 3.41...`, and since you can't run a fraction of a truck, you need at least 4.
7. **B** — real roads curve, so straight-line distance is an approximation; production systems use actual road-network or drive-time data.
8. **C** — nearest-neighbor is greedy and short-sighted, often leaving an expensive final leg because it never looks ahead.
9. **B** — savings measures exactly how many miles are avoided by pairing two stops on one truck instead of sending two separate trucks from the depot.
10. **B** — the endpoint rule and the capacity rule are both required; you can't splice into the middle of a route or exceed the vehicle's capacity.
11. **B** — more vehicles means more separate depot round trips, which adds distance even though each individual route is shorter.
12. **B** — 3 shipments is too small a sample to trust; the correct move is to flag it and gather more data, not act decisively.
13. **B** — the clause filters out carriers whose on-time percentage would be statistically unreliable due to low shipment volume.
14. **C** — LTL is the cheapest mode that still reliably beats the 5-day deadline; Intermodal is cheaper but too risky against a hard date, and Air is needlessly expensive when LTL already clears the bar.
15. **B** — cost per unit strips out volume changes, so a rising trend signals a genuine cost or efficiency problem rather than simply "we shipped more."

</details>

**Scoring:** 12+ → start Week 8. 9–11 → re-read the lecture sections behind your misses. <9 → re-read all three lectures from the top; routing and mode selection compound directly into Week 8's warehousing decisions.
