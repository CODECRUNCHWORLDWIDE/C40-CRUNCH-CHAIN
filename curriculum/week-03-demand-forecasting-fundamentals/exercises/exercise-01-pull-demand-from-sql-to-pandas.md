# Exercise 1 — Pull Demand from SQL to pandas

**Goal:** Get comfortable moving a time series from SQL into pandas and back — the round trip you'll do dozens of times this week and every week after. By the end, you have a clean, reusable `pull_demand()` function and a first look at all six SKUs' shapes.

**Estimated time:** 60 minutes.

## Setup

Confirm the seed loaded correctly:

```sql
SELECT COUNT(*) FROM demand_history;   -- must print 624
SELECT COUNT(DISTINCT sku_id) FROM demand_history;  -- must print 6
```

Install what you need if you haven't already:

```bash
pip install pandas psycopg[binary]     # Postgres
# or just use the built-in sqlite3 module — no extra install for SQLite
```

Create a file `pull_demand.py`.

## Tasks

### Task 1 — Connect and pull one SKU

Write a function that connects to your database and returns the full `demand_history` for a single SKU as a pandas `DataFrame`, sorted by date.

```python
import pandas as pd
import sqlite3  # or: import psycopg

def pull_demand(sku_id: str, db_path: str = "crunchchain.db") -> pd.DataFrame:
    conn = sqlite3.connect(db_path)
    query = """
        SELECT week_start, units_sold
        FROM demand_history
        WHERE sku_id = ?
        ORDER BY week_start
    """
    df = pd.read_sql_query(query, conn, params=(sku_id,), parse_dates=["week_start"])
    conn.close()
    return df

alpine = pull_demand("JCK-ALP-001")
print(alpine.shape)          # Expected: (104, 2)
print(alpine.head())
print(alpine.tail())
```

*(Postgres users: swap the connection for `psycopg.connect("dbname=crunchchain")` and use `%s` placeholders instead of `?`. Everything else — the DataFrame shape, the columns — is identical, which is the point: pandas doesn't care which engine produced the rows.)*

**Expected:** `(104, 2)`. `head()` starts `2023-01-02`, `tail()` ends `2024-12-23`.

### Task 2 — Set the date as a proper time index

Time-series work in pandas is much easier once `week_start` is the index, not a column.

```python
alpine = alpine.set_index("week_start")
print(alpine.index.freq)      # None — pandas doesn't know the spacing yet
alpine = alpine.asfreq("W-MON")  # tell it: weekly, Monday-anchored
print(alpine.index.freq)      # now <Week: weekday=0>
```

`asfreq` does two useful things: it declares the cadence explicitly (so later `.shift()` calls behave correctly), and it would insert a `NaN` row for any missing week — a good way to catch a data gap you didn't know about. Confirm there are none: `alpine["units_sold"].isna().sum()` should be `0`.

### Task 3 — Pull all six SKUs into one long DataFrame

Rather than six separate variables, pull everything into a single "long" DataFrame — one row per (SKU, week) — which is the shape every pandas time-series operation this week expects.

```python
def pull_all_demand(db_path: str = "crunchchain.db") -> pd.DataFrame:
    conn = sqlite3.connect(db_path)
    query = """
        SELECT sku_id, week_start, units_sold
        FROM demand_history
        ORDER BY sku_id, week_start
    """
    df = pd.read_sql_query(query, conn, parse_dates=["week_start"])
    conn.close()
    return df

all_demand = pull_all_demand()
print(all_demand.shape)          # Expected: (624, 3)
print(all_demand["sku_id"].nunique())  # Expected: 6
```

### Task 4 — Summarize each SKU (sanity-check against the lecture)

Reproduce, in pandas, the same summary the week README had you run in SQL — and confirm the numbers match exactly (they should, to the decimal — same data, same math, different tool):

```python
summary = all_demand.groupby("sku_id")["units_sold"].agg(["min", "max", "mean"]).round(1)
print(summary)
```

**Expected** (spot-check against Lecture 1's description of each SKU):

| sku_id | min | max | mean |
|---|---:|---:|---:|
| ACC-BEA-010 | 19 | 61 | 40.4 |
| BAG-DAY-020 | 52 | 96 | 72.5 |
| FLC-ZIP-030 | 39 | 126 | 81.8 |
| JCK-ALP-001 | 53 | 207 | 129.1 |
| JCK-STM-002 | 9 | 264 | 105.8 |
| SAN-TRL-040 | 0 | 119 | 53.6 |

### Task 5 — Reshape long to wide (one column per SKU)

Some operations (visual comparison, correlation checks) are easier with SKUs as columns instead of rows. Practice the reshape both ways:

```python
wide = all_demand.pivot(index="week_start", columns="sku_id", values="units_sold")
print(wide.shape)          # Expected: (104, 6)

# and back:
long_again = wide.reset_index().melt(id_vars="week_start", var_name="sku_id", value_name="units_sold")
print(long_again.shape)    # Expected: (624, 3)
```

## Done when…

- [ ] `pull_demand.py` has a working `pull_demand(sku_id)` and `pull_all_demand()`.
- [ ] `pull_demand("JCK-ALP-001").shape == (104, 2)`.
- [ ] Your six-SKU summary table matches the numbers above exactly.
- [ ] You can explain, in one sentence, why `asfreq("W-MON")` is worth calling even though the data already looked fine.
- [ ] `wide.shape == (104, 6)` and pivoting back to long recovers 624 rows.

## Stretch

- Confirm `SAN-TRL-040`'s minimum is genuinely `0` (not a data error) by printing the rows where `units_sold == 0`. How many zero weeks are there, and what time of year do they fall in?
- Write a `pull_demand_range(sku_id, start_date, end_date)` variant that filters in SQL (`WHERE week_start BETWEEN ...`) rather than pulling everything and filtering in pandas. Which approach scales better once a real company has millions of rows, and why?

## Submission

Commit `pull_demand.py` to your portfolio under `c40-week-03/exercise-01/`.
