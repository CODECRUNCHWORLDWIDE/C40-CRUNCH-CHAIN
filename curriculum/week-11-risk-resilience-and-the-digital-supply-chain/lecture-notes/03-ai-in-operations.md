# Lecture 3 — AI in Operations

> **Duration:** ~2 hours. **Outcome:** You can implement z-score and IQR anomaly detection over operating data in Python, design an exception-management rule that routes only the alerts worth a human's time, and explain precisely where agentic automation fits the plan-forecast-replan loop — and where it doesn't, yet.

Lecture 2 built the mechanism of a control tower: rolling baselines, thresholds, alerts. This lecture makes the "threshold" step statistically rigorous, then asks the question everyone eventually asks about a working alert pipeline: **can this be more automated, and how much of the response can a machine make on its own?**

## 1. From "gap" to z-score: a threshold that adapts to volatility

Lecture 2's threshold (`gap < -8`) has a flaw: it treats every metric, at every DC, as equally noisy. But Reno DC's OTIF naturally swings more day to day than Austin East's (smaller volume, more relative noise) — a fixed 8-point threshold might be too sensitive for one DC and too loose for another. The fix is the **z-score**, which measures a deviation in units of "how much this metric normally moves," not raw percentage points:

```
z = (today's value - rolling mean) / rolling standard deviation
```

```sql
SELECT ops_date, dc, otif_pct,
       ROUND(AVG(otif_pct) OVER w, 2) AS mean_14d,
       ROUND(STDDEV(otif_pct) OVER w, 2) AS stdev_14d,
       ROUND(
         (otif_pct - AVG(otif_pct) OVER w) / NULLIF(STDDEV(otif_pct) OVER w, 0)
       , 2) AS z_score
FROM daily_ops
WHERE dc = 'Austin East'
WINDOW w AS (PARTITION BY dc ORDER BY ops_date ROWS BETWEEN 13 PRECEDING AND 1 PRECEDING)
ORDER BY ops_date;
```

(SQLite has no built-in `STDDEV` — Exercise 3 and the pipeline below do this calculation in pandas instead, where it's one line: `.rolling(14).std()`.)

**Interpreting a z-score:** under a roughly normal distribution, about 95% of values fall within z = ±2, and about 99.7% fall within z = ±3. A day with `z_score = -2.4` is unusual enough to be worth a look; a day with `z_score = -0.6` is well within normal day-to-day noise and shouldn't trigger anything. This is why z-score beats a fixed raw-point threshold: **it self-calibrates to each DC's own normal volatility**, so the same rule (`|z| > 2`) is fair to a noisy DC and a stable one alike.

## 2. The other common method: IQR (interquartile range)

Z-score assumes the metric is roughly normally distributed, which mostly holds for OTIF/fill-rate data but breaks down when a metric has a hard floor, a hard ceiling, or a skewed shape. The **IQR method** makes no such assumption:

```
Q1 = 25th percentile, Q3 = 75th percentile, IQR = Q3 - Q1
Lower fence = Q1 - 1.5 × IQR
Upper fence = Q3 + 1.5 × IQR
Anything outside [lower fence, upper fence] is flagged as an outlier.
```

In pandas:

```python
import pandas as pd

df = pd.read_sql("SELECT * FROM daily_ops WHERE dc = 'Austin East' ORDER BY ops_date", conn)

q1 = df["otif_pct"].quantile(0.25)
q3 = df["otif_pct"].quantile(0.75)
iqr = q3 - q1
lower_fence = q1 - 1.5 * iqr

df["iqr_flag"] = df["otif_pct"] < lower_fence
print(f"Q1={q1:.1f}  Q3={q3:.1f}  IQR={iqr:.1f}  lower fence={lower_fence:.1f}")
print(df[df["iqr_flag"]][["ops_date", "otif_pct"]])
```

IQR is computed over the **whole series at once** (not a rolling window), which makes it better suited to a one-time "find the outliers in this dataset" pass — exactly what Challenge 2 asks for — while the rolling z-score is better suited to a **live** pipeline, because it updates its notion of "normal" every single day instead of freezing it at whatever the historical dataset looked like when you first computed the quartiles.

## 3. Exception management: not every flag deserves a human

A pipeline that flags every day with `|z| > 2` will, on pure statistics, flag roughly 1 day in 20 *even when nothing is wrong* — that's what "2 standard deviations" means. Route every one of those to a person and you get **alert fatigue**: the team starts ignoring the channel, and the one alert that matters gets lost in the noise of the ones that didn't. Good exception management adds rules on top of the raw statistical flag:

- **Persistence.** Require the anomaly to hold for 2+ consecutive days before escalating to a human — a single noisy day self-corrects; a real disruption doesn't. (Look back at this week's `daily_ops`: the Austin East OTIF dip isn't a one-day blip, it's twenty consecutive days below 90% — persistence would have confirmed it as real within 48 hours of onset, long before the trough.)
- **Severity tiers.** Not every anomaly needs a page at 2 AM. `|z| > 2` → log it, no notification. `|z| > 3` → notify the ops channel. `|z| > 3` **and** `inbound_delay_flag = TRUE` on the same day → escalate immediately, because two independent signals agreeing is much stronger evidence than one.
- **Auto-resolution.** When the metric returns inside normal range for N consecutive days, close the exception automatically — don't make a human remember to do it, and don't let a stale, already-resolved alert keep cluttering the queue.
- **Root-cause linkage.** The most useful alert isn't "OTIF is low," it's "OTIF is low **and** `inbound_delay_flag` has been TRUE for 3 straight days **and** the last PO to the flagged DC's primary CMT supplier is 12 days overdue" — connecting the anomaly to the upstream table (here, a hypothetical linked purchase-order feed from Week 6) that explains *why*, which is what actually lets a person act in minutes instead of spending an hour investigating.

## 4. Where AI in operations genuinely earns its keep

"AI in operations" covers a spectrum, and it's worth being precise about which part of that spectrum does what:

**Statistical anomaly detection** (Sections 1–2 above) — this is the workhorse. Z-scores and IQR are simple, explainable, fast, and catch the overwhelming majority of real operational anomalies. This is not a marketing claim; it's the actual mechanism behind most production control towers today, dressed up in vendor language as "AI-powered insights."

**Machine-learning anomaly detection** (isolation forests, autoencoders, clustering) — earns its keep when a *single* metric's threshold isn't the real signal, but a **combination** of several metrics moving together in an unusual pattern is. An isolation forest, for instance, can catch "OTIF is only slightly low, fill rate is only slightly low, AND lead time is only slightly up — individually none would trigger a z-score alert, but together this combination has never happened before" in a way univariate z-scores structurally cannot, because each metric is checked independently. The cost: much less explainable ("the model flagged it" is a worse answer to "why?" than "OTIF is 3.1 standard deviations below its 14-day baseline"), and it needs meaningfully more historical data to train on than a threshold rule does.

**Demand forecasting with ML** — already covered in [Week 4](../../week-04-advanced-forecasting-and-ml/); the same idea, applied upstream of operations instead of to operations data itself.

**Agentic automation in the replan loop** — the newest and most overstated layer. An "agent" here means software that can not just flag an anomaly but **take the next step**: draft a reallocation of inventory across DCs, propose an expedited shipment with a cost estimate attached, or even (in a fully automated setup) execute a small, pre-authorized replan within guardrails a human set in advance. This genuinely works today for **narrow, well-bounded, reversible** decisions — "reallocate 200 units from Memphis to Austin East, this has been done before, the cost is small and known" is a reasonable thing to let an agent draft or even execute. It does **not** yet reliably work for **novel, high-cost, hard-to-reverse** decisions — "should we qualify a new CM region in a different country" is not a decision to hand to an agent, no matter how good the underlying model is, because the cost of a bad call is large and the training data for "how do you evaluate a brand-new CM region" is thin by definition.

**The practical rule for where the line sits today:** the higher the cost of being wrong and the harder the decision is to reverse, the more a human needs to be explicitly in the loop before anything executes — not just informed after the fact. A good agentic design makes that boundary explicit in code (an approval gate, a dollar-threshold cutoff, a "this action type requires human sign-off" list) rather than leaving it to hope.

## 5. Putting it together: the anomaly-to-action pipeline

```
daily_ops (raw data)
   ↓
rolling baseline + z-score  (Lecture 2 mechanism + Section 1's statistics)
   ↓
exception rules: persistence, severity tier, root-cause linkage  (Section 3)
   ↓
   ├── low severity  → log only, no human involved
   ├── medium severity → notify ops channel, human reviews within a day
   └── high severity  → escalate now; for narrow/reversible actions, an agent
                          may draft (or, within pre-set guardrails, execute)
                          a proposed replan; a human approves anything
                          high-cost or hard to reverse
```

This is the loop Exercise 3 has you build the first two stages of, and the mini-project asks you to wire all the way through to an alert.

## 6. Check yourself

- Why is a z-score threshold generally preferable to a fixed raw-point gap threshold across DCs with different natural volatility?
- What does a z-score of -2.4 tell you, and roughly what fraction of "normal" days would you expect to see a value that extreme purely by chance?
- Name two exception-management rules that reduce alert fatigue without missing real disruptions, and explain the trade-off each one makes.
- Give one example of a decision that's a reasonable candidate for full agentic automation today, and one that isn't — and explain the distinction in terms of cost and reversibility, not just "AI isn't good enough yet."
- Why does an isolation forest (or similar multivariate method) catch some anomalies that a per-metric z-score would miss?

That closes the lecture trio. Exercise 1 scores the risk register, Exercise 2 measures this week's disruption precisely, and Exercise 3 builds the rolling-baseline pipeline this lecture's statistics feed directly into.

## Further reading

- **NIST — Guide to anomaly/intrusion detection concepts (general statistical framing, not security-specific):** <https://www.nist.gov/> (search "anomaly detection")
- **scikit-learn — Isolation Forest documentation:** <https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.IsolationForest.html>
- **pandas — `rolling()` window documentation:** <https://pandas.pydata.org/docs/reference/window.html>
- **Google Cloud — "What is an AI agent?" (vendor-neutral framing of agentic automation boundaries):** <https://cloud.google.com/discover/what-are-ai-agents>
