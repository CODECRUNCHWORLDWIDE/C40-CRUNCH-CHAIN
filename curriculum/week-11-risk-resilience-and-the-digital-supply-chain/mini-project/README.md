# Mini-Project — Stress-Test a New Disruption + Build the Replan Pipeline

> Pick a disruption that hasn't already been solved for you, model its operational impact, run it through an automated anomaly-detection pipeline, and produce the alert feed plus a costed resilience recommendation a Crunch Gear operations director would actually act on.

**Estimated time:** 2.5–3 hours, best done Saturday after the exercises and challenges.

Every other piece of this week worked against the Andes Stitch Works fire — a disruption already built into `daily_ops` for you. This project flips that: **you build the disruption dataset yourself**, from the [Week 11 README](../README.md)'s `network_risk_register`, then run it through the same detection-and-response pipeline you built in Exercise 3 and Challenge 2. This is the actual two-part job of a resilience/ops analyst: model what a real event would do to the numbers, and prove your monitoring would have caught it in time to matter.

---

## Deliverable

A directory in your portfolio `c40-week-11/mini-project/` containing:

1. `scenario_data.sql` — your disruption's `daily_ops_scenario` seed data (schema below).
2. `pipeline.py` — an adapted version of your Exercise 3 pipeline, run against your scenario data, producing an alert feed **and** a draft replan suggestion at the confirmed-alert point.
3. `report.md` — the numbers and the reasoning, structured as described in Part 3.
4. `notes.md` — a short reflection (see the end).

State which SQL engine you used (or note if you did the seed construction entirely in pandas — either is fine this week, since the table is one you're authoring, not one you're loading from `resources.md`).

---

## Part 0 — Pick your scenario (5 min)

Choose **one** of the two, both drawn from `network_risk_register`:

**Option A — Hai Phong CM Region port closure.** A typhoon closes the port for **22 days**. Because this is the *entire* import lane (not one domestic-ish CMT line), the impact is broader and the pipeline refill takes longer once the port reopens — the 34-day ocean transit time means finished goods already in the sea keep flowing, but nothing new can enter the pipeline until the port clears, so expect the visible Austin East impact to run considerably longer than the closure itself once you account for the transit-time lag on both ends.

**Option B — Austin East DC power/grid outage.** A regional grid failure fully closes Austin East for **5 days** (no shipping at all — OTIF and fill rate should be modeled as crashing to near-zero those days), followed by a shorter but real backlog-clearing recovery period as the DC catches up on the missed volume.

Either is a legitimate choice — Option A is a slower, deeper, supply-side story; Option B is a sharper, shorter, facility-side story. Pick the one you find more interesting to model.

## Part 1 — Model the disruption (60–75 min)

Create `daily_ops_scenario`, same shape as this week's `daily_ops`:

```sql
CREATE TABLE daily_ops_scenario (
    ops_id              INTEGER PRIMARY KEY,
    ops_date            DATE    NOT NULL,
    dc                  TEXT    NOT NULL,
    orders_received     INTEGER NOT NULL,
    units_shipped       INTEGER NOT NULL,
    otif_pct            NUMERIC NOT NULL,
    fill_rate_pct       NUMERIC NOT NULL,
    avg_lead_time_days  NUMERIC NOT NULL,
    inbound_delay_flag  BOOLEAN NOT NULL
);
```

1. Write **at least 45 days** of daily rows for Austin East (or the DC your scenario hits), covering: a normal baseline period, the disruption's onset and trough, and a recovery back to baseline. Base your baseline metrics on this week's real `daily_ops` figures (Austin East baseline OTIF ≈ 95.8%, fill rate ≈ 96.9%) so your scenario is grounded in the same network.
2. Your trough severity and duration are your call, but they must be **internally consistent** with your chosen event: Option A's trough should be at least as deep and considerably longer than the Andes Stitch Works case (a bigger risk, per its higher `risk_score`); Option B's trough should be sharper and shorter (near-total shutdown for a few days, faster bounce-back once power is restored, since it's a facility outage, not a supply-chain-wide sourcing gap).
3. Include at least one **other DC's** data across the same date range, showing little to no disruption — reproduce the isolation finding from Exercise 2, Task 8, for your own scenario.
4. Load it and sanity-check: `SELECT COUNT(*) FROM daily_ops_scenario;` should match the number of rows you wrote.

## Part 2 — Run the detection pipeline (60–75 min)

1. Adapt your Exercise 3 `pipeline.py` (rolling 14-day baseline, z-score, persistence rule) to run against `daily_ops_scenario` instead of `daily_ops`.
2. Report the **confirmed-detection date** — the first date your persistence rule fires — and compare it to your scenario's actual onset date. How many days of lead time would this have bought Crunch Gear before the trough?
3. **Add one new step your Exercise 3 pipeline didn't have: a draft replan suggestion.** At the confirmed-alert point, have your script print a suggested action, e.g.:
   ```
   [ALERT] Austin East OTIF confirmed anomalous on <date> (z=<value>).
   [REPLAN SUGGESTION] Recommend expediting N units via Air freight from
   <backup source> at an estimated premium of $X, pending human approval.
   ```
   You don't need real freight-rate data for this — a reasonable, stated estimate (referencing Week 7's mode-cost figures, or Challenge 1's mitigation costs, as a starting point) is enough. The point, per Lecture 3, Section 4, is that the pipeline **drafts** the action; it doesn't execute it — note explicitly in your script's output or comments that this step requires human sign-off before anything is actually booked.

## Part 3 — The report (`report.md`)

Structure it in three sections:

1. **Scenario summary** — which option you chose, the event's parameters (date, duration, affected DC), and your modeling assumptions.
2. **Detection results** — confirmed-detection date, lead time gained vs. the trough, and the draft replan suggestion your pipeline produced.
3. **Resilience recommendation** — using the `expected annual loss` framework from Challenge 1 (`likelihood × impact`, converted from your scenario's risk register row), state whether a permanent resilience investment (buffer, redundancy, or both) is worth building against this risk, with the same kind of net-benefit math Challenge 1 walked through. You can reuse Challenge 1's assumptions where they apply, or state new ones — just show your work either way.

---

## Milestones

- **Milestone 1 (60–75 min):** Part 1 — scenario data modeled and loaded.
- **Milestone 2 (60–75 min):** Part 2 — pipeline run, alert produced, replan suggestion drafted.
- **Milestone 3 (30 min):** Part 3 — assemble `report.md`, write `notes.md`.

---

## Rules

- **Your scenario data must be internally consistent** — a 5-day facility outage (Option B) producing a 40-day trough, or a 22-day port closure (Option A) recovering in 3 days, doesn't match the physics of the event you chose. State your reasoning for the shape of your curve in `report.md` if it isn't obvious.
- **The replan suggestion must explicitly note it requires human approval** — this week's Lecture 3 boundary between "draft" and "execute" is not optional framing, it's the point of the exercise.
- **No naked numbers.** Every dollar figure in `report.md` needs a one-sentence source (measured from your data, or a stated assumption).

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Scenario realism | 20% | Trough shape and duration match the chosen event's physical logic; isolation shown for the unaffected DC |
| Pipeline correctness | 25% | Rolling baseline, z-score, and persistence rule all correctly adapted from Exercise 3; confirmed-detection date is computed, not eyeballed |
| Replan suggestion | 20% | Concrete, costed, explicitly flagged as needing human approval |
| Resilience recommendation | 25% | Expected-annual-loss math shown in full, net benefit computed, a clear recommendation stated |
| Clarity | 10% | Report reads like something you'd hand to an operations director — tables, not walls of text |

---

## Reflection (`notes.md`, ~200 words)

1. Why did you pick the scenario you picked, and what was the hardest part of making its trough shape realistic?
2. How many days of lead time did your pipeline buy versus the disruption becoming "obvious" at the trough — and is that enough time to meaningfully change the outcome?
3. Where did your replan suggestion feel genuinely useful, and where did it feel like it was guessing? What data would have made it a better suggestion?
4. If Crunch Gear built the resilience investment your report recommends, what's the **next** highest-scored unmitigated risk on the register that should get this same treatment?

---

## Why this matters

This mini-project is the whole week compressed into one loop: know your risks (Lecture 1 / Exercise 1), watch continuously instead of on a fixed cycle (Lecture 2), detect statistically instead of by feel (Lecture 3 / Exercise 3 / Challenge 2), and turn a detected anomaly into a costed decision (Challenge 1's framework) — the actual plan-forecast-**replan** loop this week's learning objectives named. Keep this project; Week 12's capstone pulls every lever from the whole course, including this one, into a single network optimization.

When done: push, then take the [quiz](../quiz.md).
