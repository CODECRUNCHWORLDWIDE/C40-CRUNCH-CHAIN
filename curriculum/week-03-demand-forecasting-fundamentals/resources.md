# Week 3 — Resources

Free, public, no signup unless noted. Read the "required" set; treat the rest as reference you dip into when a specific question comes up.

## Install first

- **PostgreSQL 16+** or **SQLite 3.35+** — whichever you're using for the course; see [C33 Crunch SQL's resources](../../../C33-CRUNCH-SQL/curriculum/week-01-relational-model-and-select/resources.md) for install steps if you haven't already set one up.
- **Python 3.10+ with pandas** — `pip install pandas`. Confirm with `python3 -c "import pandas; print(pandas.__version__)"`.
- **A Postgres driver, if using Postgres from Python** — `pip install "psycopg[binary]"`. SQLite needs nothing extra; `sqlite3` ships with Python.
- No forecasting library (`statsmodels`, `prophet`, `scikit-learn`, etc.) is required this week **on purpose** — every method is built from arithmetic you write yourself, so you understand exactly what's inside it before Week 4 hands the same job to a library.

## Required reading (this week's core)

- **Hyndman & Athanasopoulos, *Forecasting: Principles and Practice* (3rd ed.) — "Time series decomposition":**
  <https://otexts.com/fpp3/decomposition.html>
  *Why: the standard, free, rigorous textbook treatment of everything in Lecture 1 — written by one of the field's most-cited authors.*
- **Hyndman & Athanasopoulos — "Some simple forecasting methods":**
  <https://otexts.com/fpp3/simple-methods.html>
  *Why: the formal treatment of naive, seasonal naive, and drift methods behind Lecture 2.*
- **Hyndman & Athanasopoulos — "Exponential smoothing":**
  <https://otexts.com/fpp3/expsmooth.html>
  *Why: SES and Holt's method, with the same notation this week's lectures use.*
- **Hyndman & Athanasopoulos — "Evaluating forecast accuracy":**
  <https://otexts.com/fpp3/accuracy.html>
  *Why: MAE, MAPE, RMSE and their known failure modes (including MAPE's zero-actual problem), from the source most forecasting practitioners cite.*

## Reference (keep in tabs)

- **PostgreSQL — Window Functions Tutorial:** <https://www.postgresql.org/docs/current/tutorial-window.html>
  *Why: `LAG`, `AVG() OVER`, and frame clauses (`ROWS BETWEEN ... PRECEDING`) power every SQL forecast this week.*
- **pandas — Time series / date functionality:** <https://pandas.pydata.org/docs/user_guide/timeseries.html>
  *Why: `asfreq`, `shift`, `rolling` — the three pandas operations you'll use in nearly every exercise.*
- **pandas — `rolling()` API reference:** <https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.rolling.html>
  *Why: exact parameters for moving-average windows, including `center=True` for the decomposition homework.*
- **NIST/SEMATECH e-Handbook — "Single Exponential Smoothing":** <https://www.itl.nist.gov/div898/handbook/pmc/section4/pmc431.htm>
  *Why: an alternative, very concise derivation of SES if Hyndman's notation doesn't click for you.*
- **NIST/SEMATECH e-Handbook — "Double Exponential Smoothing":** <https://www.itl.nist.gov/div898/handbook/pmc/section4/pmc433.htm>
  *Why: same, for Holt's method.*

## Practice beyond this week's dataset

- **M4 Competition dataset and results** — the largest public forecasting benchmark, freely downloadable: <https://github.com/Mcompetitions/M4-methods>
  *Why: the real-world version of this week's "does the fancy method actually beat the simple baseline" question, run at massive scale across 100,000 series — the empirical answer (simple methods are shockingly competitive) is exactly this week's thesis.*
- **Kaggle — "Store Item Demand Forecasting Challenge"** (free, browser-based): <https://www.kaggle.com/c/demand-forecasting-kernels-only>
  *Why: real retail demand data to practice decomposition and baseline scoring on once you're comfortable with the synthetic Crunch Gear set.*

## Deeper background (optional this week)

- **Makridakis, Spyros, "Accuracy Measures: Theoretical and Practical Concerns" (1993)** — the classic paper on why MAPE and other metrics can mislead:
  <https://doi.org/10.1016/0169-2070(93)90079-3> (paywalled; many university libraries mirror it — search the title if you have institutional access)
  *Why: the original, rigorous case for always reporting multiple error metrics together, never just one.*
- **Holt, C.C., "Forecasting seasonals and trends by exponentially weighted moving averages" (1957, reprinted 2004 in *International Journal of Forecasting*)** — the original paper behind Lecture 3's Holt's method:
  <https://doi.org/10.1016/j.ijforecast.2003.09.015>
  *Why: seeing the original 1957 derivation is a good reminder that "double exponential smoothing" is a 70-year-old idea, not a recent invention — and still a strong baseline today.*

## Glossary

| Term | Definition |
|------|------------|
| **Level** | The current baseline magnitude of a demand series — "roughly how much per period, right now." |
| **Trend** | A sustained, one-directional drift in the level over many periods, with no calendar reset. |
| **Seasonality** | A pattern that repeats on a fixed calendar cycle and returns to the same relative position each cycle. |
| **Noise / residual** | Whatever remains after level, trend, and seasonality are removed — the unpredictable part. |
| **Additive decomposition** | `y = level + trend + seasonal + noise`; seasonal swing stays a roughly constant number of units. |
| **Multiplicative decomposition** | `y = level × trend × seasonal × noise`; seasonal swing stays a roughly constant percentage. |
| **Holdout** | Recent periods deliberately withheld from model fitting, used only to score forecast accuracy honestly. |
| **Horizon (h)** | How many periods ahead a forecast is made for. |
| **Naive forecast** | `ŷ_{t+1} = y_t` — tomorrow looks like today. |
| **Seasonal naive forecast** | `ŷ_t = y_{t-period}` — this period looks like the same period one cycle ago. |
| **Moving average forecast** | The mean of the last `k` periods, used as the forecast for the next period. |
| **Data leakage** | Accidentally including the value you're forecasting inside the inputs used to forecast it — always check your window/shift logic. |
| **α (alpha)** | Exponential smoothing's level-smoothing parameter; higher = more reactive, lower = more stable. |
| **β (beta)** | Holt's method's trend-smoothing parameter; higher = trend estimate adapts faster to recent changes. |
| **MAE** | Mean Absolute Error — average size of the miss, in original units. |
| **MAPE** | Mean Absolute Percentage Error — average miss as a percentage of the actual; undefined when actual = 0. |
| **RMSE** | Root Mean Squared Error — like MAE but squares errors first, so large misses count disproportionately more. |
| **Bias** | The signed (not absolute) average error; positive = systematic under-forecasting, negative = systematic over-forecasting. |

---

*Broken link? Open an issue or PR.*
