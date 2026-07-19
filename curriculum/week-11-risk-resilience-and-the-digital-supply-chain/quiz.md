# Week 11 — Quiz

Fifteen questions. Lectures closed. Aim for 12/15 before moving to Week 12. A mix of multiple-choice and short "what would you compute" questions — the answer key at the bottom explains the *why*, not just the letter.

---

**Q1.** In the `expected annual loss = likelihood × impact` framework, a risk with likelihood 2 (≈15%/yr) and impact $400,000 has roughly the same expected annual loss as a risk with:

- A) Likelihood 5 (≈90%/yr) and impact $400,000
- B) Likelihood 4 (≈65%/yr) and impact $90,000
- C) Likelihood 1 (≈5%/yr) and impact $1,200,000
- D) Likelihood 2 (≈15%/yr) and impact $40,000

---

**Q2.** A **single point of failure (SPOF)** is best defined as:

- A) Any node that has ever caused a problem in the past
- B) The most expensive node in the network
- C) A node or lane whose loss stops or badly degrades a function the network has no other way to perform
- D) Any supplier located outside the home country

---

**Q3.** In Crunch Gear's `network_risk_register`, `Hai Phong CM Region` scores highest (20) because:

- A) It has the highest `annual_volume_share_pct` of any row
- B) It combines high likelihood (frequent typhoon-season closures) with high impact (severe if it happens), and it's a SPOF
- C) It's the only row classified as `Geopolitical`
- D) It's the most recently added row to the register

---

**Q4.** On the likelihood × impact matrix, a risk with **low likelihood and high impact** (e.g., likelihood 2, impact 5) typically calls for which resilience approach?

- A) Ignore it — low likelihood means it's not worth tracking
- B) Insure or transfer it — rare but severe risks are often cheaper to transfer than to engineer around
- C) Build permanent redundant capacity regardless of cost
- D) Treat it identically to a high-likelihood, low-impact risk

---

**Q5.** Which of the four resilience levers (buffers, redundancy, flexibility, visibility) is the one that **reduces the time before a disruption is noticed**, without reducing the disruption's underlying severity?

- A) Buffers
- B) Redundancy
- C) Flexibility
- D) Visibility

---

**Q6.** Why does a **monthly-average** operating report tend to understate a disruption that both begins and partially resolves within that same month?

- A) Monthly reports round all numbers to the nearest whole percent
- B) Averaging blends the disruption's worst days together with weeks of normal operation, diluting the signal until the average barely looks abnormal
- C) Monthly reports only track cost, not service level
- D) It's a database bug specific to `GROUP BY MONTH`

---

**Q7.** In a rolling-baseline query (`AVG(otif_pct) OVER (... ROWS BETWEEN 13 PRECEDING AND 1 PRECEDING)`), the window deliberately **excludes the current row**. Why?

- A) SQL requires it for syntax reasons
- B) Including today's value in its own baseline would dilute an anomaly by mixing it into the average meant to detect it
- C) It makes the query run faster
- D) It has no effect either way; excluding the current row is just a style convention

---

**Q8.** A `MATERIALIZED VIEW` in PostgreSQL differs from a plain `VIEW` mainly because:

- A) A materialized view stores its result physically and must be refreshed on a schedule, while a plain view recomputes on every query
- B) A materialized view can only be used with `SELECT *`
- C) A plain view is faster in every case
- D) Materialized views don't support `JOIN`

---

**Q9.** A z-score of **-2.4** for today's OTIF value means:

- A) OTIF dropped by exactly 2.4 percentage points from yesterday
- B) Today's value is 2.4 standard deviations below the rolling baseline mean — unusual, but not yet at the ~99.7% extreme of a 3-sigma event
- C) 2.4% of all historical days had a lower OTIF
- D) The pipeline detected a data-entry error

---

**Q10.** The **IQR (interquartile range)** anomaly-detection method differs from a rolling z-score mainly in that IQR:

- A) Requires machine learning
- B) Is computed once over the whole series (a static snapshot), while a rolling z-score updates its notion of "normal" every day
- C) Only works on integer data
- D) Cannot be computed in Python

---

**Q11.** Why does adding a **persistence rule** (require an anomaly to hold for 2+ consecutive days before escalating) reduce alert fatigue without materially delaying real detection, in this week's Austin East scenario?

- A) It doesn't — persistence always delays every alert by exactly 2 days
- B) A single noisy day tends to self-correct, while a real disruption (like the Andes Stitch Works fire's aftermath) shows up on consecutive days — so persistence filters isolated statistical noise while still confirming genuine, multi-day anomalies quickly
- C) Persistence rules only work with SQLite, not PostgreSQL
- D) It converts the z-score calculation into an IQR calculation

---

**Q12.** According to this week's framing of agentic automation in the replan loop, which decision is the **best current candidate** for an agent to draft or even execute within pre-set guardrails?

- A) Whether to qualify an entirely new offshore contract-manufacturing region
- B) Reallocating 200 units of known SKUs between two DCs, a routine and reversible action with a small, well-understood cost
- C) Renegotiating Crunch Gear's master carrier contracts
- D) Deciding whether to discontinue a product line

---

**Q13.** Crunch Gear's `daily_ops` table shows Austin East's OTIF dropping sharply starting 2026-04-27, even though the triggering event (the Andes Stitch Works fire) happened on 2026-04-15. The roughly 12-day gap between the event and its visible impact is best explained by:

- A) A data-entry delay in the `daily_ops` table
- B) Austin East's existing safety-stock buffer absorbing the shortfall for about 12 days before it was exhausted
- C) The fire actually happened on 2026-04-27, not 2026-04-15
- D) OTIF is reported on a 12-day lag by definition

---

**Q14.** In Exercise 2, Memphis DC and Reno DC showed **no material change** in OTIF or fill rate during the Andes Stitch Works disruption window, while Austin East collapsed. This is best explained by:

- A) Memphis and Reno simply had better management during that period
- B) The disruption was a single point of failure specific to a supplier that only Austin East's product line depended on — proof that a SPOF, by definition, damages only the part of the network that depends on it
- C) The data for Memphis and Reno was recorded incorrectly
- D) Andes Stitch Works also supplies Memphis and Reno, so this result is unexpected and unexplained

---

**Q15.** Challenge 1's Risk A (Andes Stitch Works dual-sourcing) came out with a **negative net benefit** under the given assumptions ($9,526/yr expected-loss reduction against an $18,000/yr mitigation cost), while Risk B (Hai Phong buffer) came out positive. The correct takeaway is:

- A) Negative net benefit always means "never mitigate this risk under any circumstances"
- B) The dollar math doesn't support the specific mitigation as costed, but non-dollar factors (contractual penalties, reputational damage, correlated risk) not captured in the model could still justify it — the number is an input to the decision, not the whole decision
- C) The calculation must contain an error, since resilience investments should always show a positive net benefit
- D) Risk A should be removed from the register entirely since it isn't worth fixing

---

## Answer key

<details>
<summary>Reveal after attempting</summary>

1. **C** — `0.15 × 400,000 = 60,000`; `0.05 × 1,200,000 = 60,000` — same expected annual loss despite very different likelihood/impact shapes. (Check the others: A = 360,000; B = 58,500 ≈ close but not exact; D = 6,000 — C is the cleanest match.)
2. **C** — the defining test is "if this went to zero tomorrow, is there a Plan B, or would we be improvising?"
3. **B** — likelihood 4 × impact 5 = 20, and it's flagged `single_point_of_failure = TRUE`, all three factors driving the top score.
4. **B** — rare-but-severe risks are the classic insurance/transfer case; engineering around every low-likelihood risk is usually not cost-effective.
5. **D** — visibility shrinks detection time, not the disruption's underlying severity; buffers/redundancy/flexibility address severity directly.
6. **B** — averaging a bad week into three normal weeks produces a monthly figure that looks only mildly off, hiding the real severity of the bad days.
7. **B** — including today's anomalous value in its own baseline average would pull the baseline down too, shrinking the very gap the query is trying to measure.
8. **A** — a materialized view is a stored, schedule-refreshed result; a plain view recomputes its underlying query every time it's referenced.
9. **B** — z = -2.4 is a meaningful deviation (beyond the ~95% two-sigma band) but not yet at the ~99.7% three-sigma extreme.
10. **B** — IQR is typically computed once over an entire historical series; a rolling z-score recomputes its baseline continuously as new data arrives.
11. **B** — isolated noisy days rarely repeat two days running, while a genuine disruption's effects compound and persist, which is exactly what the persistence rule is built to distinguish.
12. **B** — narrow, low-cost, easily reversible actions are the current sweet spot for agentic automation; the other three options are high-cost, hard-to-reverse, strategic decisions that still need a human fully in the loop.
13. **B** — safety stock is designed to absorb exactly this kind of gap for a while; the visible impact begins once that buffer runs out, not the moment the upstream event occurs.
14. **B** — a SPOF's damage is scoped to whatever depends on it; Memphis and Reno's product flow didn't depend on Andes Stitch Works, so they were structurally insulated from this particular disruption.
15. **B** — a negative net benefit under stated assumptions is a real, useful finding, not an error — it means the dollar case alone doesn't clear the bar, and a full recommendation has to weigh what the model didn't capture.

</details>

**Scoring:** 12+ → start Week 12. 9–11 → re-read the lecture sections behind your misses. <9 → re-read all three lectures from the top; risk scoring, disruption costing, and the detection pipeline all feed directly into the Week 12 capstone.
