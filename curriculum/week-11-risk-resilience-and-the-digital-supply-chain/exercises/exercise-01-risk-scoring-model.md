# Exercise 1 — Build a Risk-Scoring Model

**Goal:** Turn the 16-row `network_risk_register` into the ranked, quadrant-classified view a resilience review actually works from — and produce a defensible "mitigate first" shortlist.

**Estimated time:** 90 minutes.

## Setup

Confirm the seed is loaded:

```sql
SELECT COUNT(*) FROM network_risk_register;   -- must print 16
```

Create `solutions.sql` and put each answer under a `-- Task N` comment.

## Tasks

1. **Every risk, scored.** Select `node_name`, `risk_category`, `likelihood`, `impact`, and a computed `risk_score` (`likelihood * impact`), sorted by `risk_score` descending. *(Expected: `Hai Phong CM Region` is #1 with a score of 20.)*

2. **The SPOF count.** Count how many rows have `single_point_of_failure = TRUE`. *(Expected: 11 of 16.)*

3. **SPOFs with no mitigation.** Select every row where `single_point_of_failure = TRUE` **and** `mitigation_status = 'None'`, sorted by `risk_score` descending. These are the risks with zero backup plan today. *(Expected: 5 rows — confirm `Hai Phong CM Region` and `Austin East DC` are both in this list.)*

4. **Quadrant classification.** Add a `CASE`-derived `quadrant` column using this rule: `likelihood >= 3 AND impact >= 3` → `'Mitigate now'`; `likelihood >= 3 AND impact < 3` → `'Monitor/absorb'`; `likelihood < 3 AND impact >= 3` → `'Insure/transfer'`; otherwise → `'Accept'`. Show `node_name`, `likelihood`, `impact`, `quadrant`, sorted by quadrant. *(Expected: 6 rows land in `'Mitigate now'`, including two — `Memphis DC` and `Austin East Regional Labor Market` — that are **not** single points of failure, proof the quadrant and the SPOF flag are independent signals.)*

5. **Count by category.** `GROUP BY risk_category`, showing `COUNT(*)` and `AVG(likelihood * impact)` per category, sorted by average score descending. *(Expected: `Geopolitical` has the highest average score of the six categories, at 16.0 — driven by just 2 rows, both severe.)*

6. **The "must mitigate first" shortlist.** Combine Tasks 3 and 4's logic into one query: every risk where `quadrant = 'Mitigate now'` **and** `mitigation_status != 'Mitigated'`, sorted by `risk_score` descending. This is the list a resilience review should walk through first. *(Expected: 6 rows — in this register, every "Mitigate now" risk still lacks full mitigation, so this list matches Task 4's exactly.)*

7. **Volume-weighted risk.** For rows where `annual_volume_share_pct IS NOT NULL`, compute `risk_score * annual_volume_share_pct / 100.0 AS volume_weighted_score` — a rough proxy for "risk score, scaled by how much of the network actually rides on this node." Sort descending. Does the ranking change much from Task 1's raw score? Which node moves up or down the most, and why does that make business sense?

8. **Mitigation coverage.** `GROUP BY mitigation_status`, showing `COUNT(*)` and `SUM(risk_score)` (compute `risk_score` inline) per status. What share of *total* risk score across the whole register currently has `mitigation_status = 'None'`?

## Expected result (spot checks)

- Task 1 → `Hai Phong CM Region`, score 20, ranks #1.
- Task 2 → 11 SPOFs out of 16.
- Task 3 → 5 unmitigated SPOFs.
- Task 4 → 6 rows in `'Mitigate now'`.
- Task 6 → 6 rows (same set as Task 4 — every one of them still lacks full mitigation).

## Done when…

- [ ] `solutions.sql` has all 8 queries under `-- Task N` comments.
- [ ] Your Task 1 and Task 6 rankings match the spot checks above.
- [ ] You can name, from memory, the top 3 risks on the "must mitigate first" shortlist and their scores.
- [ ] Task 7's answer includes one sentence on why volume-weighting changed (or didn't change) the ranking.

## Stretch

- Add a third dimension to the score — `detectability` (1 = you'd notice within hours, 5 = you might not notice for weeks) — assign a plausible 1–5 value to each of the 16 risks yourself, and recompute a 3-factor score (`likelihood * impact * detectability`, the classic FMEA "Risk Priority Number" shape). Does `Order Management System` move up the list? It should — a cyber outage with no monitoring in place is exactly the kind of risk that's easy to underrate on a 2-factor matrix because "impact" alone doesn't capture how long it might run undetected.
- Cross-reference Task 6's shortlist against Week 6's supplier scorecard concepts: for `Andes Stitch Works` and `ButtonWorks Supply` (both on the shortlist), what scorecard metric from Week 6 would have given an early warning sign that these suppliers were a widening risk, independent of a formal risk register?

## Submission

Commit `solutions.sql` to your portfolio under `c40-week-11/exercise-01/`.
