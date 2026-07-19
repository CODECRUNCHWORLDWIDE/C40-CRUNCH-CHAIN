# Exercise 3 — Automate a Daily Ops Pipeline

**Goal:** Build the first two stages of a real control-tower pipeline — a rolling 14-day baseline and a z-score anomaly flag — over `daily_ops`, then add the persistence rule from Lecture 3 that turns raw statistical noise into a trustworthy signal.

**Estimated time:** 60 minutes.

## Setup

Confirm the seed is loaded:

```sql
SELECT COUNT(*) FROM daily_ops;   -- must print 180
```

This exercise is **Python-first**. SQLite has no built-in `STDDEV`, and even on PostgreSQL, a persistence rule (checking whether *yesterday* was also flagged) is far more natural as a pandas column shift than a second layer of window functions. Pull the whole table into a DataFrame once, then work in pandas:

```python
import sqlite3       # or: import psycopg2 / your Postgres driver of choice
import pandas as pd

conn = sqlite3.connect("crunch_chain_wk11.db")
df = pd.read_sql("SELECT * FROM daily_ops ORDER BY dc, ops_date", conn)
df["ops_date"] = pd.to_datetime(df["ops_date"])
```

Create `pipeline.py` and build it up task by task.

## Tasks

1. **Rolling baseline, per DC.** For each `dc`, compute a 14-day rolling mean and rolling standard deviation of `otif_pct`, using only the **prior** 14 days (not including today). In pandas, `.shift(1)` before `.rolling(14)` does exactly this:

   ```python
   df = df.sort_values(["dc", "ops_date"])
   df["baseline_mean"] = df.groupby("dc")["otif_pct"].transform(lambda s: s.shift(1).rolling(14).mean())
   df["baseline_std"]  = df.groupby("dc")["otif_pct"].transform(lambda s: s.shift(1).rolling(14).std())
   ```

   *(Expected: the first 14 rows per DC have `NaN` baselines — there isn't 14 prior days of history yet. That's correct, not a bug.)*

2. **Z-score.** Compute `z = (otif_pct - baseline_mean) / baseline_std` for every row with a valid baseline.

3. **Raw flag.** Add a boolean column `z_flag = z < -2`. Filter to `dc == 'Austin East'` and print every flagged row's date and z-score. *(Expected: **10 flagged days**, starting with an isolated single day on **2026-04-18** — note this one is a false-positive-shaped early blip, not the real disruption yet — followed by more flags clustering from **2026-04-26** onward.)*

4. **Persistence rule.** A single flagged day isn't enough to escalate (Lecture 3, Section 3). Add a column `confirmed = z_flag AND z_flag.shift(1)` (today **and** yesterday both flagged). Find the **first date** where `confirmed` is `True` for Austin East. *(Expected: **2026-04-29** — note this is 11 days *after* the noisy 2026-04-18 false alarm, and still a full 5-6 days *before* the trough begins on 2026-05-04, which is real, actionable lead time.)*

5. **False-positive rate check.** Run the same raw `z_flag` logic (Task 3) against `Memphis DC` and `Reno DC` — the two DCs that were **not** meaningfully disrupted. Count flagged days for each. *(Expected: Memphis DC ≈ 2 flagged days, Reno DC ≈ 4 flagged days, out of 46 scoreable days each — roughly matching the "1 in 20 by pure chance" rate Lecture 3 predicts for a `|z| > 2` threshold.)* Confirm neither DC produces a `confirmed` (2-consecutive-day) flag — the persistence rule should filter out this normal statistical noise entirely.

6. **Build the alert feed.** Produce a DataFrame (or a printed table) of every `confirmed = True` row across all three DCs, with columns `dc`, `ops_date`, `otif_pct`, `baseline_mean`, `z`. This is what a real alert channel or exceptions table would receive.

7. **Compare to the naive baseline.** In 2-3 sentences, compare Task 4's 2026-04-29 detection date to the monthly-cycle scenario from Lecture 2 (where April's average OTIF looked unremarkable and the real damage wouldn't have been visible until an early-May report). How many days of lead time did the daily pipeline buy?

## Expected result (spot checks)

- Task 3 → 10 flagged days for Austin East, first one 2026-04-18 (a false alarm).
- Task 4 → confirmed (2-day persistence) flag first fires 2026-04-29.
- Task 5 → Memphis DC ≈2, Reno DC ≈4 raw flags; zero `confirmed` flags for either.

## Done when…

- [ ] `pipeline.py` runs top to bottom without errors and prints the Task 6 alert feed.
- [ ] Your Task 4 confirmed-flag date matches the spot check.
- [ ] You can explain, in one sentence, why the persistence rule is necessary given Task 5's false-positive counts.
- [ ] Task 7's comparison states a specific number of lead-time days gained.

## Stretch

- Wrap the whole thing in a function `run_daily_pipeline(df, dc, z_threshold=-2, persistence_days=2)` that returns the alert feed for any DC/threshold/persistence combination. Re-run with `z_threshold=-1.5` and `persistence_days=3` and see how the detection date and false-positive count both move — this is the exact trade-off Lecture 2 described between "too sensitive" and "too loose."
- Add a print statement that would function as the "notification" step of the pipeline — a formatted string like `[ALERT] Austin East OTIF confirmed anomalous on 2026-04-29 (z=-4.57, 2 consecutive days below baseline)`. This is literally what a Slack-webhook or email-alert integration would send in a production version of this pipeline (out of scope to actually wire up this week, but the string is the whole payload).

## Submission

Commit `pipeline.py` to your portfolio under `c40-week-11/exercise-03/`.
