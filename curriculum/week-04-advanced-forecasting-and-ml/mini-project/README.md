# Mini-Project — Backtested ML Forecaster vs. the Week 3 Baseline

> One question, answered with evidence: does a tuned, backtested gradient-boosted tree ensemble actually beat the simple baseline from Week 3 — or does the simple model still win? Both outcomes are a legitimate, fundable answer. What isn't legitimate is answering without a rolling-origin backtest to back it up.

**Estimated time:** 4 hours, best split across Thursday/Friday after the exercises and challenges.

This is the week's capstone, and it's deliberately built to resist the trap every forecasting team eventually falls into: falling in love with a fancier model because it *feels* more rigorous, without ever proving — on a backtest that can't leak — that it actually forecasts better than the boring thing a junior analyst could ship in an afternoon. You will build both, backtest both fairly on identical folds, and let the numbers decide. If the ML model doesn't win, your job is to say so clearly and explain why, not to keep tuning until it does.

---

## Part 0 — Seed the data (skip if already loaded)

This is the same `weekly_demand` table used across every lecture, exercise, and challenge this week. If you already have it loaded from earlier in the week, skip to Part A.

`generate_demand.py` — fully deterministic (seed = 40), builds 3 SKUs x 2 regions x 156 weeks (3 years) of weekly demand with realistic trend, seasonality, and promo/price structure:

```python
import numpy as np, pandas as pd

rng = np.random.default_rng(40)
N = 156
weeks = pd.date_range("2024-01-01", periods=N, freq="W-MON")

def make_series(base, trend, amp, phase, noise_sd, promo_weeks, promo_lift):
    t = np.arange(N)
    seasonal = amp * np.sin(2 * np.pi * (t - phase) / 52.0)
    level = base + trend * t + seasonal
    promo = np.zeros(N)
    promo[promo_weeks] = promo_lift
    y = level + promo + rng.normal(0, noise_sd, N)
    return np.clip(np.round(y), 0, None).astype(int)

rows = []
for region, mult in [("Northeast", 1.0), ("West", 0.8)]:
    # JCK-100 Alpine Shell Jacket — winter-peaking, trended, promo-responsive
    pw = rng.choice(N, size=20, replace=False)
    y = make_series(180 * mult, 0.35 * mult, 70 * mult, 0, 14, pw, 45 * mult)
    price = np.full(N, 129.0); price[pw] = 99.0
    flag = np.zeros(N, dtype=bool); flag[pw] = True
    for i, wk in enumerate(weeks):
        rows.append(("JCK-100", region, wk.date().isoformat(), int(y[i]), float(price[i]), bool(flag[i]), bool(wk.month == 12)))

    # TNT-220 Trailhead Tent 2P — summer-peaking, promo-responsive
    pw2 = rng.choice(N, size=16, replace=False)
    y2 = make_series(90 * mult, 0.15 * mult, 55 * mult, 26, 10, pw2, 30 * mult)
    price2 = np.full(N, 189.0); price2[pw2] = 159.0
    flag2 = np.zeros(N, dtype=bool); flag2[pw2] = True
    for i, wk in enumerate(weeks):
        rows.append(("TNT-220", region, wk.date().isoformat(), int(y2[i]), float(price2[i]), bool(flag2[i]), bool(wk.month == 12)))

    # SPR-901 Repair Patch Kit — intermittent, low volume
    lam = 2.2 * mult
    nz = rng.random(N) < 0.35
    y3 = np.where(nz, rng.poisson(lam, N) + 1, 0)
    for i, wk in enumerate(weeks):
        rows.append(("SPR-901", region, wk.date().isoformat(), int(y3[i]), 14.0, False, bool(wk.month == 12)))

df = pd.DataFrame(rows, columns=["sku", "region", "week_start", "units", "unit_price", "promo_flag", "holiday_flag"])
df.to_csv("weekly_demand.csv", index=False)
print(df.shape)   # (936, 7)
```

Load it into your database of choice:

**PostgreSQL:**
```bash
createdb crunch_chain
python3 generate_demand.py
psql crunch_chain -c "
CREATE TABLE weekly_demand (
    sku          TEXT NOT NULL,
    region       TEXT NOT NULL,
    week_start   DATE NOT NULL,
    units        INTEGER NOT NULL,
    unit_price   NUMERIC NOT NULL,
    promo_flag   BOOLEAN NOT NULL,
    holiday_flag BOOLEAN NOT NULL,
    PRIMARY KEY (sku, region, week_start)
);"
psql crunch_chain -c "\COPY weekly_demand FROM 'weekly_demand.csv' WITH (FORMAT csv, HEADER true);"
```

**SQLite (pandas does the table creation for you):**
```python
import sqlite3, pandas as pd
df = pd.read_csv("weekly_demand.csv")
con = sqlite3.connect("crunch_chain.db")
df.to_sql("weekly_demand", con, if_exists="replace", index=False)
```

Sanity check — should print `936`:
```sql
SELECT COUNT(*) FROM weekly_demand;
```

---

## Part A — Build the ML forecaster (90 min)

Pick **two** `(sku, region)` pairs to forecast — at minimum one high-volume, clearly-seasonal series (`JCK-100` or `TNT-220`, either region) and, if you want the extra challenge, `SPR-901` (know going in: a tree ensemble on a 36%-nonzero series is a rough fit — Croston from Challenge 1 would normally be the right call there; including it is optional and explicitly for the stretch goal, not required).

For each series:

1. Build the lag/rolling/calendar feature matrix from Lecture 3 §1 — reuse your Exercise 3 code.
2. Tune `GradientBoostingRegressor`'s `n_estimators`, `max_depth`, and `learning_rate` using your **rolling-origin backtest itself** as the tuning signal (never a single train/test split — that would just reintroduce the leakage problem one level up, by overfitting hyperparameters to one lucky split).
3. Record your final chosen hyperparameters and a one-sentence justification for each in `report.md`.

---

## Part B — Build the Week 3 baseline, fairly (30 min)

The baseline must be **at least as strong as what a competent analyst would actually ship** — not a strawman. Use the better of these two, chosen *per series* based on which the data actually calls for (state which you picked and why):

- **Seasonal naive** (`lag_52`) — the right default when the series has strong, stable year-over-year seasonality and little trend.
- **4-week moving average** — the right default when the series is closer to flat/noisy without strong seasonality.

Do not let the baseline be a strawman you picked to lose — that defeats the entire purpose of this mini-project. If you're unsure which is fairer for a given series, backtest both and report both; use whichever wins as "the baseline" you're trying to beat.

---

## Part C — Backtest both, on identical folds, honestly (60 min)

Using your hand-written `rolling_origin_folds` function from Exercise 3 (reuse it — don't rebuild it):

1. Run **at least 4 folds** (or as many as your training-history floor allows — this dataset supports 3–4 with a 13-week horizon and a 60-week minimum training window, same as the lectures).
2. For every fold, record MAE **and mean signed error (bias)** for both the ML model and the baseline. Bias matters as much as MAE here — Lecture 3 §5 showed gradient-boosted trees can systematically underforecast a still-trending series' peaks; you need to know if that's happening to *your* series, not just assume it.
3. Produce a results table: `sku, region, fold, model, mae, bias`.

---

## Part D — The verdict (45 min)

In `report.md`, for **each** of your two series, answer explicitly:

1. **Did the ML model beat the baseline on mean MAE across folds?** By how much, in absolute units and as a percentage of the series' average weekly volume?
2. **Was the win (or loss) consistent across folds**, or did the ranking flip? (Lecture 3 §6.)
3. **Did either model show a directional bias** — and if the ML model underforecast systematically on a trending series, say so plainly; don't bury it.
4. **Your recommendation**: for *this specific series*, would you actually deploy the ML forecaster in production, or is the added complexity, retraining burden, and monitoring overhead not worth the margin over the simple baseline? A correct, well-argued "the baseline wins, ship it instead" is scored identically to a correct, well-argued "the ML model wins, ship it" — the rubric rewards the *quality of the argument*, not which model comes out on top.

---

## Deliverable

A directory in your portfolio `c40-week-04/mini-project/` containing:

1. `generate_demand.py` (or a note that you reused the shared one, with any changes noted).
2. `features.py` — your lag/rolling/calendar feature-engineering code, reused from Exercise 3.
3. `backtest.py` — the rolling-origin backtest loop producing the `results.csv` table (sku, region, fold, model, mae, bias).
4. `results.csv` — the raw fold-by-fold numbers.
5. `report.md` — Parts A's hyperparameter choices, Part B's baseline choice and justification per series, and Part D's four-question verdict per series.

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| ML feature engineering | 15% | Correct lags/rolling stats, `.shift(1)` used correctly, no leakage |
| Fair baseline construction | 15% | Baseline choice justified per series, not a strawman |
| Backtest correctness | 25% | Rolling-origin folds correctly built, reused from Exercise 3, both models scored on identical folds |
| Bias reported, not just MAE | 15% | Mean signed error computed and discussed per fold, trend-extrapolation bias (§5) checked explicitly |
| Verdict quality | 20% | All four questions answered per series, with numbers backing every claim |
| Honesty | 10% | If the baseline wins on a series, the report says so plainly rather than reframing the result |

---

## Stretch goals

- Add `SPR-901` as a third series and compare the ML model, the Week 3 baseline, **and** Challenge 1's Croston forecast on the same backtest folds (adapted for a rate-based forecast). Which wins, and does your Challenge 1 conclusion about MAE being a poor metric for intermittent demand hold up here too?
- Combine the ML model and Holt-Winters into a simple blended forecast (average of the two, or a weighted average favoring whichever historically has lower bias) for your trending series, and check whether the blend fixes the extrapolation-bias problem from Lecture 3 §5 without giving up the ML model's overall MAE advantage.
- Extend Part C to 6+ folds by shrinking `fold_size` to 8 weeks instead of 13. Does the model ranking hold at a shorter forecast horizon?

## Why this matters

This is the real job: someone will eventually ask you to justify why a more complex forecasting approach is worth the engineering and operational cost over what's already running — and "it has lower training error" or "it's more sophisticated" is not an acceptable answer. A rolling-origin backtest with honest bias reporting, on identical folds, is the actual bar a production forecasting change has to clear. Keep this mini-project's backtest harness (`features.py`, `backtest.py`) — Week 5 forecasts feed directly into safety-stock and reorder-point decisions, and you'll want the same rigor there.

When done: push, then take the [quiz](../quiz.md) and start [Week 5 — Inventory: EOQ, reorder point, safety stock, service level](../../week-05-inventory-eoq-reorder-point-and-safety-stock/).
