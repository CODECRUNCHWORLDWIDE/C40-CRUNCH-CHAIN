# Mini-Project — Forecast 50 SKUs and Score Every One Against a Naive Baseline

> Scale everything you built this week from 6 SKUs to 50. Forecast each one, score every method honestly, and ship an accuracy report a demand-planning manager could actually read and act on.

**Estimated time:** 3 hours, best done Saturday after the exercises and challenges.

This is the week's capstone, and it's deliberately closer to a real job than anything before it: you don't get to hand-pick one clean SKU and admire your work. You get 50, all different, and your job is to run the same rigorous process across every one of them, then summarize the results so someone who has never heard of MAPE can still make a decision from your report.

---

## Step 0 — Generate the 50-SKU demand history (SQL + Python, never a spreadsheet)

You already have 6 SKUs (`JCK-ALP-001`, `JCK-STM-002`, `ACC-BEA-010`, `BAG-DAY-020`, `FLC-ZIP-030`, `SAN-TRL-040`) loaded from the week README. This mini-project needs 44 more, for 50 total. Rather than hand-typing another 4,500+ `INSERT` rows, generate them with a documented, **seeded** (fully reproducible) Python script and load the result straight into your database — the same SQL-and-Python-only discipline this course uses everywhere, just automated once you're past the "learn every row by hand" stage of the week.

Create `generate_50_skus.py`:

```python
import random, math, datetime
import sqlite3  # or: import psycopg

CATEGORIES = {
    # category: (level range, trend/wk range, seasonal-amplitude-as-%-of-level range, peak ISO weeks, noise-as-%-of-level range)
    "Jackets":     ((60, 180),  (-0.3, 0.4),  (0.35, 0.65), [1, 2, 51, 52], (0.07, 0.13)),
    "Fleece":      ((40, 110),  (-0.1, 0.6),  (0.10, 0.30), [10, 15, 40, 45], (0.06, 0.11)),
    "Accessories": ((20, 70),   (-0.15, 0.15),(0.15, 0.35), [1, 2, 51], (0.08, 0.14)),
    "Bags":        ((40, 100),  (-0.1, 0.2),  (0.0, 0.12),  [20, 25, 30], (0.09, 0.14)),
    "Footwear":    ((30, 100),  (-0.4, 0.2),  (0.35, 0.70), [26, 27, 28], (0.08, 0.14)),
}
NAMES = {
    "Jackets": ["Ridge Parka","Storm Shell","Glacier Coat","Tundra Jacket","Peak Windbreaker",
                "Cascade Anorak","Summit Vest","Blizzard Bomber","Frost Shell","Cirque Jacket"],
    "Fleece": ["Trail Fleece","Basecamp Pullover","Ridgeline Quarter-Zip","Alpine Cardigan",
               "Wander Fleece Vest","Timberline Half-Zip","Highline Fleece","Crestline Pullover"],
    "Accessories": ["Trail Gloves","Ridge Scarf","Summit Cap","Cascade Gaiters","Alpine Buff",
                     "Trek Mittens","Ridge Beanie","Wander Neck Gaiter"],
    "Bags": ["Commuter Tote","Trail Duffel","Summit Backpack","Weekender Bag","Camera Sling",
             "Hydration Pack","Trail Waist Pack","Overnight Duffel"],
    "Footwear": ["Canyon Sandal","Trail Runner","Basin Slide","River Shoe","Summer Moc",
                 "Delta Sandal","Camp Slip-On","Trailhead Flip"],
}
PREFIX = {"Jackets": "JCK", "Fleece": "FLC", "Accessories": "ACC", "Bags": "BAG", "Footwear": "SAN"}
START_NUM = {"Jackets": 3, "Fleece": 31, "Accessories": 11, "Bags": 21, "Footwear": 41}  # continues after the 6 core SKUs

random.seed(4050)  # fixed seed — everyone who runs this gets IDENTICAL data, so results are comparable

def generate_sku_dim(n_new: int = 44):
    skus = []
    counter = dict(START_NUM)
    name_idx = {k: 0 for k in NAMES}
    cats_cycle = list(CATEGORIES.keys())
    for i in range(n_new):
        cat = cats_cycle[i % len(cats_cycle)]
        sku_id = f"{PREFIX[cat]}-{counter[cat]:03d}"
        counter[cat] += 1
        base_names = NAMES[cat]
        n = name_idx[cat]
        name = base_names[n % len(base_names)] + (" II" if n >= len(base_names) else "")
        name_idx[cat] += 1
        lvl_r, trend_r, amp_r, peaks, noise_r = CATEGORIES[cat]
        level = round(random.uniform(*lvl_r), 1)
        trend = round(random.uniform(*trend_r) / 10.0, 3)
        amp = round(level * random.uniform(*amp_r), 1)
        peak = random.choice(peaks)
        noise_sd = round(level * random.uniform(*noise_r), 1)
        skus.append((sku_id, name, cat, level, trend, amp, peak, noise_sd))
    return skus

def generate_demand(skus, start=datetime.date(2023, 1, 2), weeks=104):
    rows = []
    for sku_id, name, cat, level, trend, amp, peak, noise_sd in skus:
        for w in range(weeks):
            d = start + datetime.timedelta(weeks=w)
            wk = d.isocalendar()[1]
            seasonal = amp * math.cos(2 * math.pi * (wk - peak) / 52.0)
            base = level + trend * w + seasonal
            units = max(0, round(base + random.gauss(0, noise_sd)))
            rows.append((sku_id, d.isoformat(), units))
    return rows

if __name__ == "__main__":
    skus = generate_sku_dim()
    demand_rows = generate_demand(skus)

    conn = sqlite3.connect("crunchchain.db")
    conn.executemany(
        "INSERT INTO sku_dim (sku_id, sku_name, category) VALUES (?, ?, ?)",
        [(s[0], s[1], s[2]) for s in skus],
    )
    conn.executemany(
        "INSERT INTO demand_history (sku_id, week_start, units_sold) VALUES (?, ?, ?)",
        demand_rows,
    )
    conn.commit()
    conn.close()
    print(f"Loaded {len(skus)} new SKUs, {len(demand_rows)} new demand rows.")
```

Run it once:

```bash
python3 generate_50_skus.py
```

Then confirm you're at 50 SKUs and the expected row count:

```sql
SELECT COUNT(*) FROM sku_dim;          -- 50
SELECT COUNT(*) FROM demand_history;   -- 624 + 44*104 = 5,200
```

**Why generate rather than hand-type 5,200 rows:** this is exactly how real synthetic/benchmark datasets get built on the job — a documented, seeded script, never a spreadsheet, never manual entry. The fixed `random.seed(4050)` means your 50-SKU catalog is bit-for-bit identical to anyone else's who runs this script, so your accuracy numbers are comparable across the whole cohort even though nobody typed the data by hand.

---

## Deliverable

A directory in your portfolio `c40-week-03/mini-project/` containing:

1. `generate_50_skus.py` — the script above (or your own equivalent, if you changed parameters — document any changes).
2. `forecast_all.py` — pulls all 50 SKUs, builds naive, seasonal naive, moving-average (k=4), single exponential smoothing, and Holt's double exponential smoothing for each, scored on the **last 12 weeks** as holdout.
3. `accuracy_report.csv` (or `.md` table) — one row per SKU: `sku_id`, `category`, best-performing method, that method's MAE/RMSE/MAPE/bias, and the **naive baseline's** MAE/RMSE/MAPE/bias for comparison.
4. `report.md` — a written summary (see "What the report must answer," below).

Everything runs against the SQL tables from Step 0. Note which engine (PostgreSQL or SQLite) you used.

---

## Requirements

1. **Every SKU gets every method.** Don't hand-pick a subset — build naive, seasonal naive, MA4, SES, and Holt for all 50 SKUs, in a loop, the way Exercise 2's Task 5 set you up to do.
2. **One consistent holdout.** Use the last 12 weeks (index 92–103, `2024-10-07` through `2024-12-23`) for every SKU, same as every lecture and exercise this week. Consistency here is what makes cross-SKU comparison meaningful.
3. **Score against a naive baseline, always.** For every SKU, your report must show the naive baseline's error *next to* your best method's error — never report a method's accuracy in isolation. That comparison is the entire thesis of this week.
4. **Handle MAPE's zero-actual problem.** Some Footwear SKUs will have holdout weeks near or at zero, same as `SAN-TRL-040` in the core data. Your MAPE computation must exclude zero-actual weeks and **report how many rows were excluded, per SKU**, in the accuracy report — silently skewed MAPE numbers are not acceptable this week.
5. **Pick a "best method" per SKU, systematically — not by eye.** For each SKU, choose whichever of the five methods has the lowest MAE on that SKU's own holdout (or justify a different tie-break rule if you use one, e.g., preferring the cheaper method within 5% of the best MAE).
6. **No spreadsheets.** All 50 SKUs' data lives in SQL; all scoring happens in pandas. If you find yourself wanting to eyeball-sort a spreadsheet of 50 rows, that's a `pandas.DataFrame.sort_values()` call instead.

---

## What the report (`report.md`) must answer

Write roughly 400–600 words, backed by numbers from `accuracy_report.csv`:

1. **Which method won most often across the 50 SKUs?** Count wins per method (a simple `value_counts()` on your "best method" column). Does the answer match what Lecture 1's category descriptions would predict — e.g., do Jackets mostly favor seasonal naive, do Bags mostly favor naive/MA4?
2. **How much did the winning methods actually beat naive by, on average?** Compute the average percentage improvement in MAE over the naive baseline, across all 50 SKUs. Is the improvement large and consistent, or small and inconsistent? What does that tell you about how much effort a real ops team should spend tuning forecasts vs. just shipping naive/seasonal-naive everywhere?
3. **Which SKUs, if any, had NO method beat naive?** Flag them by name. A SKU where nothing beats naive is not a failure of your code — it's a legitimate finding (this SKU's demand may be closer to a true random walk) and should be reported as such, not hidden.
4. **How many SKUs had MAPE-excluded rows, and what's your recommendation for those SKUs** — report MAE/RMSE instead of leaning on MAPE, exclude and disclose, or something else?
5. **One paragraph, written for a VP:** if Crunch Gear's demand-planning team could only adopt ONE non-naive method across their whole catalog next quarter (not the ability to hand-pick per SKU — a real operational constraint many teams actually have), which would you recommend, and why, given what you found across all 50 SKUs?

---

## Rubric

| Criterion | Weight | "Great" looks like |
|-----------|------:|--------------------|
| Correctness | 30% | All 50 SKUs scored, all 5 methods, consistent 12-week holdout |
| Naive-benchmark discipline | 20% | Every reported "best method" number sits next to naive's number for the same SKU |
| MAPE handling | 15% | Zero-actual weeks excluded and the exclusion count disclosed per SKU |
| Report clarity | 20% | `report.md` answers all 5 questions with numbers, in prose a non-technical reader could act on |
| Reproducibility | 15% | `generate_50_skus.py` runs cleanly, the seed is unchanged (or the change is documented), row counts match |

---

## Stretch goals

- Group the accuracy report by `category` (from `sku_dim`) instead of just by SKU. Does one category (Jackets? Footwear?) systematically favor seasonal naive while another (Bags?) systematically favors naive/MA4? Report the pattern.
- For the 3–5 SKUs with the *worst* best-method MAPE, look at their generating parameters (amplitude, noise) — can you explain, from the parameters alone, why they were hard to forecast? (This is only possible because you generated the data yourself and know the ground truth — a real job almost never gives you this luxury, which is worth a sentence of reflection in `report.md`.)
- Re-run the whole pipeline with a different `random.seed(...)` and see how much your headline numbers ("average % improvement over naive," "which method wins most") move. If they move a lot, what does that tell you about how much you should trust a forecasting benchmark built from a single random dataset — in this course or on the job?

---

## Why this matters

Fifty SKUs is small by real-catalog standards (a mid-size retailer forecasts thousands), but it's exactly big enough to break the temptation to eyeball a single chart and declare victory. This is the shape of a real demand-planning deliverable: not "here's a cool forecast," but "here's a scored, benchmarked, honestly-reported comparison across the whole catalog, with the naive baseline sitting right next to every claim." Keep `accuracy_report.csv` — you'll extend this exact scoring framework in Week 4 when backtesting and ML methods enter the comparison.

When done: push, then take the [quiz](../quiz.md).
