# Challenge 1 — Resilience Network Redesign

**Time:** ~90 minutes. **Difficulty:** Medium-hard. **Judgment matters as much as arithmetic.**

## The scenario

Crunch Gear's leadership has read Exercise 1's "must mitigate first" shortlist and Exercise 2's real cost figure for the Andes Stitch Works fire, and they've asked you for one thing: **for the two highest-scored unmitigated risks, tell us whether fixing them is actually worth the money.** Not "resilience is good" — an actual net-benefit number for each, using the `expected annual loss = likelihood × impact` framework from Lecture 1.

You'll cost two different resilience levers against two different risks — **redundancy** against a supplier risk, **buffer** against a facility/geopolitical risk — because the right lever depends on the risk, and leadership wants to see you reason about both.

## Given assumptions

**Converting the register's 1–5 likelihood score to an annual probability** (use this table for both risks):

| Likelihood score | Approx. probability of ≥1 event/year |
|---|---|
| 1 | 5% |
| 2 | 15% |
| 3 | 35% |
| 4 | 65% |
| 5 | 90% |

## Part 1 — Baseline expected annual loss (20 min)

For each of the two risks below, pull `likelihood` from `network_risk_register`, convert it using the table above, and compute `expected_annual_loss = probability × impact_per_event`.

**Risk A — Andes Stitch Works** (sole CMT line, Outerwear). Impact-per-event: use Exercise 2's Task 7 measured total (**≈$49,482** — the real cost of the fire scenario you already simulated).

**Risk B — Hai Phong CM Region** (sole offshore CM region + import gateway, [Week 7](../../week-07-logistics-and-transportation-analytics/)'s HPH-ATX lane). Impact-per-event: **$145,000** (given — a region-wide port closure is larger in scope than a single supplier's fire, since it can't be bridged with a short domestic air-freight hop the way Andes Stitch Works' gap was; it hits the entire 8,100-mile import lane).

Report both baseline expected annual losses.

## Part 2 — Design and cost a mitigation for each (45 min)

**Risk A mitigation — redundancy (dual-sourcing).** Crunch Gear dual-sources **40% of Outerwear CMT volume** to **Pacific Rim Garments** (already qualified, [Week 6](../../week-06-procurement-and-supplier-analytics/)). Ongoing cost: **$18,000/year** (reserved-capacity price premium plus the annual cost of keeping a second line qualified and audited). Assume this cuts the impact-per-event of a future Andes Stitch Works outage by **55%** (a working backup absorbs most of the volume, so both the depth and duration of the service-level dip shrink). Compute the new impact-per-event, the new expected annual loss, and the **net benefit** (expected-loss reduction minus mitigation cost).

**Risk B mitigation — buffer (safety stock).** Size a safety-stock buffer for Hai Phong-sourced finished Outerwear at Austin East, using the [Week 5](../../week-05-inventory-management-eoq-and-safety-stock/) safety-stock formula that accounts for variability in **both** demand and lead time:

```
SS = z × sqrt(avg_lead_time × stddev_demand² + avg_demand² × stddev_lead_time²)
```

Inputs:
- Average daily demand: **220 units/day**, standard deviation of daily demand: **45 units**
- Average lead time: **34 days** (Hai Phong → Austin East ocean transit, per Week 7), standard deviation of lead time: **4 days**
- Target service level: **98%** → `z = 2.05`

Compute the required safety-stock units. Then, at a per-unit cost of **$58** and an annual carrying-cost rate of **25%**, compute the annual carrying cost of holding that buffer. Assume a buffer of this size cuts the impact-per-event of a future Hai Phong disruption by **50%**. Compute the new impact-per-event, the new expected annual loss, and the net benefit.

## Part 3 — Recommend (15 min)

Write `challenge-01.md` with:

1. A table: Risk, baseline expected annual loss, mitigation cost, post-mitigation expected annual loss, net benefit.
2. A clear **do this / don't do this (yet)** recommendation for each risk, backed by the net-benefit number.
3. If either net benefit came out negative, don't just say "skip it" — name one non-dollar factor (contractual OTIF penalties not captured in the $42/line assumption, reputational damage with the wholesale account from Exercise 1, regulatory exposure, etc.) that could still justify the spend, and say whether you think it plausibly does.

## Constraints

- Show every step of the arithmetic — a bare final number with no visible calculation is not a complete submission.
- Use the given assumptions as stated; don't substitute your own impact or cost figures without flagging clearly that you did and why.
- The two risks use two **different** resilience levers on purpose (redundancy vs. buffer) — don't default to "just add safety stock" for both; explain in one sentence why each lever fits its risk better than the other lever would.

## Hints

<details>
<summary>On the safety-stock formula</summary>

This is the exact formula from Week 5's Lecture on safety stock under joint demand and lead-time variability. Compute the term inside the square root first (`avg_lead_time × stddev_demand²` plus `avg_demand² × stddev_lead_time²`), take the square root, then multiply by `z`. Sanity check: the result should be in the range of roughly 1,500–2,200 units — if you get something wildly outside that, check you squared the standard deviations and not the raw values.

</details>

<details>
<summary>On why Risk A might come out net-negative</summary>

It's a legitimate, defensible result if your Risk A net benefit comes out negative under these assumptions — that's not a mistake to "fix" by changing the given numbers. A moderate-likelihood, moderate-impact risk with an expensive redundancy fix is exactly the kind of risk the Lecture 1 matrix says to treat with more nuance than "always mitigate." The interesting part of this challenge is reasoning about what to do when the dollar math says "skip it" but other considerations might not.

</details>

## How success is judged

| Signal | Weak submission | Strong submission |
|---|---|---|
| Baseline math | Skips the probability conversion or uses impact alone | Correct `probability × impact` for both risks, using the given table |
| Safety-stock calc | Wrong formula or unsquared standard deviations | Formula applied correctly, result in the sane range, carrying cost computed from it |
| Net-benefit math | Missing or inconsistent units ($/year vs. one-time) | Every number is clearly annualized and the final net-benefit subtraction is shown |
| Recommendation | "Mitigate everything" or "skip everything" | A specific, numbers-backed call per risk, including honest handling of a negative net benefit |

## Submission

Commit `challenge-01.md` (with supporting `.sql`/`.py` if you used it for the arithmetic) to your portfolio under `c40-week-11/challenge-01/`.
