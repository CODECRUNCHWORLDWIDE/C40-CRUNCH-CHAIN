# Lecture 3 — Carrier Selection & Freight Analytics

> **Duration:** ~2 hours. **Outcome:** You can score every carrier Crunch Gear uses on cost and on-time reliability with a single SQL query, reason about the cost/speed/reliability trade-off between modes, and produce a freight-spend summary a finance partner would trust.

Lecture 1 picked the mode. Lecture 2 picked the route. This lecture picks **who drives the truck** — and then steps back to look at the whole freight bill the way a controller would: where is the money going, and is it buying the service level it should?

## 1. On-time performance — the metric that actually matters

A shipment is **on time** when it arrives by (or before) its promised transit day:

```sql
SELECT shipment_id, carrier, promised_transit_days, actual_transit_days,
       CASE WHEN actual_transit_days <= promised_transit_days
            THEN 1 ELSE 0 END AS on_time
FROM shipments
LIMIT 10;
```

Aggregate that per carrier and you get an **on-time percentage** — the single number every shipper tracks first:

```sql
SELECT carrier,
       COUNT(*)                                                     AS n_shipments,
       SUM(CASE WHEN actual_transit_days <= promised_transit_days
                THEN 1 ELSE 0 END)                                  AS n_on_time,
       ROUND(100.0 * SUM(CASE WHEN actual_transit_days <= promised_transit_days
                              THEN 1 ELSE 0 END) / COUNT(*), 1)     AS on_time_pct
FROM shipments
GROUP BY carrier
ORDER BY on_time_pct DESC;
```

```
 carrier                 | n_shipments | n_on_time | on_time_pct
--------------------------+-------------+-----------+-------------
 Yellowline Freight       |          16 |        16 |       100.0
 RailBridge Intermodal    |           6 |         6 |       100.0
 SkyBridge Air Cargo      |           3 |         3 |       100.0
 Basecamp Carriers        |          18 |        17 |        94.4
 SwiftPost                |          18 |        17 |        94.4
 RangeExpress              |          15 |        14 |        93.3
 Longhaul Truckload       |          19 |        17 |        89.5
 CrossLTL                 |           7 |         6 |        85.7
 Nimbus Parcel            |          19 |        16 |        84.2
 Pacific Rim Ocean Lines  |           3 |         1 |        33.3
```

**Read the sample sizes before you read the percentage.** Pacific Rim Ocean Lines looks catastrophic at 33.3% — but that's 1 late shipment out of 3. SkyBridge Air Cargo looks perfect at 100% — also on just 3 shipments. Neither number is trustworthy yet; six weeks of Ocean/Air data (a lane used roughly every three weeks) just hasn't accumulated enough volume to separate "genuinely bad carrier" from "bad luck once." Contrast that with Nimbus Parcel's 84.2% on **19** shipments — that's a real signal, not noise. **A rule of thumb worth keeping:** don't act decisively on a reliability metric backed by fewer than ~10–15 observations; flag it as "watch" instead of "cut."

## 2. Cost — total spend and cost per pound, together

On-time percentage alone doesn't tell you if a carrier is a good deal. Pair it with spend:

```sql
SELECT carrier, mode,
       COUNT(*)                                    AS n,
       ROUND(SUM(freight_cost), 2)                 AS total_spend,
       ROUND(SUM(freight_cost) / SUM(weight_lbs), 4) AS cost_per_lb
FROM shipments
GROUP BY carrier, mode
ORDER BY total_spend DESC;
```

This is the table a freight-spend review actually runs on: which carrier/mode combination is costing the most, and is that because they move the most freight (fine) or because they charge more per pound for the same job (worth a conversation). `Longhaul Truckload` and `Basecamp Carriers` will show up near the top of `total_spend` mostly *because* they carry the heavy FTL freight — that's expected and not a red flag by itself. The red flag is two carriers running the **same mode on the same lane** at meaningfully different `cost_per_lb`.

## 3. A combined carrier scorecard

Put reliability and cost side by side, and you have a real scorecard:

```sql
SELECT carrier,
       COUNT(*)                                                       AS n_shipments,
       ROUND(100.0 * SUM(CASE WHEN actual_transit_days <= promised_transit_days
                              THEN 1 ELSE 0 END) / COUNT(*), 1)       AS on_time_pct,
       ROUND(SUM(freight_cost), 2)                                    AS total_spend,
       ROUND(AVG(freight_cost / NULLIF(weight_lbs,0)), 4)             AS avg_cost_per_lb
FROM shipments
GROUP BY carrier
HAVING COUNT(*) >= 10          -- drop carriers we don't have enough data on yet
ORDER BY on_time_pct DESC, avg_cost_per_lb ASC;
```

The `HAVING COUNT(*) >= 10` line is doing real work — it's the SQL expression of the "don't trust a percentage from 3 shipments" rule from Section 1. Filtering low-volume carriers out of the *ranking* (while still reporting on them separately, low-volume-but-flagged) is standard practice in freight analytics.

## 4. The cost / speed / reliability trade-off

Every mode-and-carrier decision is really a three-way trade-off, and you can never maximize all three at once:

| You want... | You'll usually pay in... |
|---|---|
| **Lowest cost** (Ocean, full FTL trucks, slow LTL) | Speed — and sometimes reliability, if the cheap carrier is cheap *because* it's less consistent |
| **Fastest transit** (Air, expedited Parcel) | Cost — often by 10-40x per pound over the cheapest option |
| **Highest reliability** | Sometimes cost, sometimes nothing at all — some carriers are simply better-run for the same price, which is exactly what the scorecard in Section 3 is built to surface |

The mistake to avoid is treating this as "cheap vs. good" — the scorecard above shows `Yellowline Freight` and `RailBridge Intermodal` at **100% on-time**, which means for those lanes, reliability was free; there was no trade-off to make at all, just a better vendor. The trade-off only bites when the cheapest option and the most reliable option are genuinely different carriers — which is most of the time, but not always. Never assume the trade-off exists before you've checked.

### A worked mode-selection example

Crunch Gear needs 2,000 lb of Alpine Shell Jackets at the Reno DC in 5 days for a flash sale that starts Friday. Three options exist on that lane:

| Option | Mode | Cost | Transit |
|---|---|---|---|
| A | Intermodal | ~$130 (at $0.065/lb) | 3–7 days (variable) |
| B | LTL | ~$540 (at $0.27/lb) | 2 days |
| C | Air | ~$7,340 (at $3.67/lb) | 4 days |

Option A is cheapest but its transit variance (3–7 days) makes it a real risk against a hard Friday deadline — if it lands on day 7, the flash sale opens with no inventory, and lost-sale cost from a blown launch very likely exceeds the ~$400 saved over LTL. Option B (LTL, 2 days, low variance) comfortably beats the deadline with margin and costs a fraction of Air. Option C is wildly overpriced for a 5-day-out deadline it doesn't need to hit that fast. **The right call is B** — not because it's cheapest, and not because it's fastest, but because it's the cheapest option that reliably clears the actual constraint. This is the exact reasoning [Challenge 2](../challenges/challenge-02-mode-selection-tradeoff.md) has you apply across a batch of real shipment scenarios.

## 5. Freight spend analysis — the view finance wants

Tie cost back to volume shipped and you get freight spend **as a rate**, which is the number that actually tracks efficiency over time (raw total spend just tracks how much you shipped):

```sql
SELECT
    origin,
    ROUND(SUM(freight_cost), 2)                                AS total_freight_spend,
    SUM(units_shipped)                                          AS total_units,
    ROUND(SUM(freight_cost) / NULLIF(SUM(units_shipped),0), 3)  AS freight_cost_per_unit
FROM shipments
GROUP BY origin
ORDER BY total_freight_spend DESC;
```

`freight_cost_per_unit` is the number to watch quarter over quarter: if it's rising while `total_units` holds steady, something got more expensive — a carrier rate hike, a shift toward smaller/less-efficient shipments, a fuel surcharge — and it's worth a `GROUP BY mode` or `GROUP BY carrier` drill-down (exactly the queries from Sections 1–3) to find out which.

## 6. Check yourself

- Why is a carrier's on-time percentage misleading when it's built on fewer than ~10 shipments?
- Write the `HAVING` clause you'd add to exclude any carrier with fewer than 15 shipments from a ranking.
- In the worked mode-selection example, why was the *cheapest* option not the right call, and why was the *fastest* option not the right call either?
- What's the difference between "total freight spend" and "freight cost per unit," and why does a finance partner usually care more about the second?
- A carrier has the lowest average cost per pound on a lane but the worst on-time percentage. What's the one additional number you'd want before recommending a switch away from them?

If those are automatic, you're ready for this week's exercises — computing cost per lane, building a nearest-neighbor route, and scoring every carrier from scratch.

## Further reading

- **Council of Supply Chain Management Professionals — Annual State of Logistics Report** (free executive summary): <https://cscmp.org/CSCMP/Educate/SCM_Definitions_and_Glossary_of_Terms.aspx>
- **PostgreSQL — Aggregate Functions:** <https://www.postgresql.org/docs/current/functions-aggregate.html>
- **PostgreSQL — `HAVING` clause reference (in "The SQL Language" tutorial):** <https://www.postgresql.org/docs/current/tutorial-agg.html>
