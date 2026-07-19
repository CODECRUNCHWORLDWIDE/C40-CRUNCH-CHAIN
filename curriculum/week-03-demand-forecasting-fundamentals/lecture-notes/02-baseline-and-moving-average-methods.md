# Lecture 2 — Baselines & Moving Average Methods

> **Duration:** ~2 hours. **Outcome:** You can build naive, seasonal naive, and moving-average forecasts straight from a SQL query into pandas, understand exactly what each one assumes about the world, and — using real numbers from the Alpine Shell Jacket — see for yourself why "the simple one" sometimes beats "the one that looks at more data."

Before you reach for anything fancier, you build three forecasts that require no library, no training, and almost no code. They are not toys. In real operations teams, a seasonal naive forecast is often what actually ships for slow-moving or highly seasonal SKUs, because it's transparent, cheap to compute, and — as you're about to see — hard to beat when the seasonal pattern is strong and stable. Every method after this one has to prove it's worth its added complexity by beating these three on held-out data. That's not a suggestion; it's the rule this whole course follows starting today.

## 1. The forecasting setup, precisely

A forecast made at time `t` for a future period `t+h` is written `ŷ_{t+h|t}` ("y-hat for `t+h`, given information up to `t`"). `h` is the **horizon** — how many periods ahead. This week we forecast one week ahead (`h=1`) unless stated otherwise; the same logic extends to any horizon.

To honestly evaluate a method, you need a **holdout**: weeks you pretend you don't know yet, so you can compare the forecast to what actually happened. Pull the Alpine Shell Jacket series and set aside the **last 12 weeks** (index 92–103, `2024-10-07` through `2024-12-23`) as holdout for every worked example in this lecture and the next:

```sql
SELECT week_start, units_sold
FROM demand_history
WHERE sku_id = 'JCK-ALP-001'
ORDER BY week_start;
```

You'll load this into pandas in Exercise 1. For now, work through the mechanics on the printed numbers below — they're the real last 16 weeks of this series, unedited:

| week_start | units_sold |
|------------|-----------:|
| 2024-09-09 | 105 |
| 2024-09-16 | 126 |
| 2024-09-23 | 120 |
| 2024-09-30 | 107 |
| **2024-10-07** | **138** ← holdout starts here |
| 2024-10-14 | 142 |
| 2024-10-21 | 153 |
| 2024-10-28 | 150 |
| 2024-11-04 | 164 |
| 2024-11-11 | 162 |
| 2024-11-18 | 165 |
| 2024-11-25 | 187 |
| 2024-12-02 | 176 |
| 2024-12-09 | 193 |
| 2024-12-16 | 207 |
| 2024-12-23 | 197 |

This is the ramp into the winter peak — demand climbing fast, week over week. Keep that in mind; it's exactly the condition that will separate these methods.

## 2. Naive forecast

**The rule:** tomorrow looks like today. `ŷ_{t+1} = y_t`.

That's the entire model. No parameters, no history beyond the single most recent observation.

```sql
-- one-step-ahead naive forecast: this week's forecast = last week's actual
SELECT
    week_start,
    units_sold,
    LAG(units_sold, 1) OVER (ORDER BY week_start) AS naive_forecast
FROM demand_history
WHERE sku_id = 'JCK-ALP-001'
ORDER BY week_start;
```

`LAG(units_sold, 1)` pulls the previous row's value into the current row — exactly what "forecast = last actual" means in SQL. The window function does in one line what a loop would take five lines to do in pandas, though you'll do it both ways in the exercises.

**When naive is a genuinely good choice:** demand with no strong trend and no strong seasonality — a "random walk" series where the best guess for next period really is this period. It's also the *mandatory comparison point* for every other method: if your fancy model can't beat naive, it isn't adding value, it's adding complexity.

**When naive fails:** obviously, any series with real trend (naive always lags a trend by exactly one period) or real seasonality (naive completely ignores the calendar — it would forecast June's low season using December's peak-season last value).

## 3. Seasonal naive forecast

**The rule:** this period looks like the same period, one full cycle ago. For weekly data with annual seasonality, that's 52 weeks back: `ŷ_t = y_{t-52}`.

```sql
SELECT
    week_start,
    units_sold,
    LAG(units_sold, 52) OVER (ORDER BY week_start) AS seasonal_naive_forecast
FROM demand_history
WHERE sku_id = 'JCK-ALP-001'
ORDER BY week_start;
```

Same `LAG` function, just offset by 52 instead of 1 — the SQL is nearly identical, which is a nice illustration of how similar these "simple" methods really are under the hood.

Seasonal naive requires at least one full cycle of history before it can produce a forecast at all — that's why this week's seed loads **two years**, not one: you need last year's same week to exist before you can look it up. This is a real operational constraint, not just a teaching convenience — it's exactly why a brand-new SKU (no history yet) can't use seasonal naive on day one, a problem you'll deal with directly in Challenge 1.

**When seasonal naive shines:** any series with a strong, *stable* seasonal pattern year over year — which describes most of Crunch Gear's jacket and sandal lines. It automatically captures the calendar effect that plain naive completely misses.

**When seasonal naive fails:** if the seasonal pattern itself is shifting (a promo moved from November to October this year), or if there's a strong trend layered on top (seasonal naive by itself ignores growth — last year's December doesn't know this year's business grew 20%).

## 4. Moving-average forecast

**The rule:** forecast the average of the last `k` periods. `ŷ_{t+1} = (y_t + y_{t-1} + ... + y_{t-k+1}) / k`.

`k` (the **window**) is the one parameter you choose. A small `k` (e.g., 4 weeks) reacts quickly to recent changes but is noisy; a large `k` (e.g., 13 weeks, a quarter) is smoother but slower to notice a real shift.

```sql
-- 4-week moving average forecast (uses the 4 weeks BEFORE the current one — no leakage)
SELECT
    week_start,
    units_sold,
    AVG(units_sold) OVER (
        ORDER BY week_start
        ROWS BETWEEN 4 PRECEDING AND 1 PRECEDING
    ) AS ma4_forecast
FROM demand_history
WHERE sku_id = 'JCK-ALP-001'
ORDER BY week_start;
```

Read the frame clause carefully: `ROWS BETWEEN 4 PRECEDING AND 1 PRECEDING` — four rows *before* the current one, stopping one row before it. That's deliberate. If you wrote `ROWS BETWEEN 3 PRECEDING AND CURRENT ROW`, you'd be averaging in the very value you're trying to forecast — a classic **data-leakage bug** that makes a backtest look far better than the method will actually perform live, because in production you obviously don't have this week's actual yet when you're forecasting this week.

**The core weakness of moving average: it always lags a trend.** A `k`-period moving average is mathematically guaranteed to lag a steadily rising series by roughly `(k+1)/2` periods worth of trend. The bigger your window, the smoother your forecast, but also the further behind reality it runs during any sustained climb or drop.

## 5. Worked comparison: all three, on the real ramp

Using the last-16-weeks table above, here's the 4-week moving average forecast for each of the 12 holdout weeks, computed exactly as the SQL above would produce it:

| week_start | actual | MA4 forecast | error (actual − MA4) |
|------------|-------:|---------------:|----------------------:|
| 2024-10-07 | 138 | 114.5 | +23.5 |
| 2024-10-14 | 142 | 122.8 | +19.2 |
| 2024-10-21 | 153 | 126.8 | +26.2 |
| 2024-10-28 | 150 | 135.0 | +15.0 |
| 2024-11-04 | 164 | 145.8 | +18.2 |
| 2024-11-11 | 162 | 152.2 | +9.8 |
| 2024-11-18 | 165 | 157.2 | +7.8 |
| 2024-11-25 | 187 | 160.2 | +26.8 |
| 2024-12-02 | 176 | 169.5 | +6.5 |
| 2024-12-09 | 193 | 172.5 | +20.5 |
| 2024-12-16 | 207 | 180.2 | +26.8 |
| 2024-12-23 | 197 | 190.8 | +6.2 |

Every single error is **positive** — the MA4 forecast under-shoots on every one of the 12 weeks. That's the lag effect from Section 4, made visible: demand is climbing into the winter peak, and a 4-week trailing average is always looking slightly into the past relative to where the series actually is. Mean absolute error (you'll compute this formally next lecture) works out to **17.2 units**.

Now the same 12 weeks, naive and seasonal naive:

| Method | Mean absolute error (12-week holdout) |
|--------|---------------------------------------:|
| Moving average (k=4) | 17.2 |
| Naive (last value) | 11.8 |
| **Seasonal naive (t−52)** | **7.7** |

**Naive beats moving average here**, because a single last-value forecast, while still lagging the trend, doesn't smear in three extra weeks of "even more stale" data the way MA4 does. And **seasonal naive beats both**, because last year's same week already "knows" that the winter ramp happens — it's not lagging the current trend at all, it's referencing the correct season directly. The lesson: during a period of strong, predictable seasonal movement, a method that ignores the calendar (moving average, naive) will structurally underperform one that respects it (seasonal naive) — no amount of tuning `k` fixes that, because the problem isn't noise, it's the wrong information source.

This is exactly the kind of result a forecaster needs to *see*, not just be told. You'll reproduce this comparison yourself on all six SKUs in Exercise 2 and Exercise 3, and you'll find it doesn't hold everywhere — the Daypack, which has no seasonality, will tell a completely different story. That's the whole point: no single baseline wins everywhere, which is exactly why Challenge 1 asks you to choose per SKU instead of picking one method for the whole catalog.

## 6. Combining methods: weighted moving average (preview)

One quick extension worth knowing about, even though we don't dwell on it this week: a **weighted moving average** assigns more weight to recent periods instead of treating all `k` periods equally —

```sql
SELECT
    week_start,
    units_sold,
    (
        0.4 * LAG(units_sold, 1) OVER (ORDER BY week_start) +
        0.3 * LAG(units_sold, 2) OVER (ORDER BY week_start) +
        0.2 * LAG(units_sold, 3) OVER (ORDER BY week_start) +
        0.1 * LAG(units_sold, 4) OVER (ORDER BY week_start)
    ) AS wma4_forecast
FROM demand_history
WHERE sku_id = 'JCK-ALP-001'
ORDER BY week_start;
```

Weights (0.4, 0.3, 0.2, 0.1) sum to 1.0 — that's a requirement, or your forecast will be systematically scaled wrong. This reduces (but doesn't eliminate) the lag problem, because more-recent, more-relevant weeks count for more. It's a natural bridge to next lecture's topic: exponential smoothing is, mathematically, a weighted average of *every* past observation, with weights that decay smoothly the further back you go — instead of you hand-picking four weights that sum to one, the smoothing parameter picks an infinite, automatically-decaying set of weights for you.

## 7. Check yourself

- Write, in words, the naive and seasonal-naive forecast rules without looking back.
- Why does seasonal naive need a full cycle (52 weeks) of history before it can forecast anything?
- In the `ROWS BETWEEN 4 PRECEDING AND 1 PRECEDING` frame clause, what would change — and why would it be a bug — if you wrote `AND CURRENT ROW` instead?
- Why does a 4-week moving average lag a rising trend, mechanically? (Hint: what's the average age, in weeks, of the four observations it's built from?)
- On the Alpine Jacket's 12-week winter-ramp holdout, which method won, and in one sentence, why?
- Name a demand shape (think about the six SKUs) where you'd expect moving average to *beat* seasonal naive instead.

If those are automatic, Lecture 3 adds exponential smoothing — a way to weight recent history without hand-picking a window — and gives you the formal error metrics (MAE, MAPE, RMSE, bias) to score every method you've built so far.

## Further reading

- **PostgreSQL — Window Functions Tutorial:** <https://www.postgresql.org/docs/current/tutorial-window.html>
- **PostgreSQL — Window Function Calls (frame clause reference, `ROWS BETWEEN`):** <https://www.postgresql.org/docs/current/sql-expressions.html#SYNTAX-WINDOW-FUNCTIONS>
- **Hyndman & Athanasopoulos, *Forecasting: Principles and Practice* — "Some simple forecasting methods":** <https://otexts.com/fpp3/simple-methods.html>
