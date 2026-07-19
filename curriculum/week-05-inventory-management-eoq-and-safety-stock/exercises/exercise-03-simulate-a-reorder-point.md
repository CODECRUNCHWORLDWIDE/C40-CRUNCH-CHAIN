# Exercise 3 — Simulate a Reorder-Point Policy

**Goal:** Stop trusting the safety-stock formula purely on faith. Simulate a full year of random daily demand under an (s,Q) policy, measure the *realized* cycle-service level and fill rate, and see for yourself that the formula's 95% target is a real, empirically-verifiable promise — with real sampling noise around it.

**Estimated time:** 1.5 hours.

## Why simulate at all?

Lecture 2's `SS = z*σ_DLT` formula is a probability statement: *if* demand during lead time really is normal with the assumed mean and standard deviation, *then* you'll stock out in about 5% of cycles at a 95% target. A simulation lets you generate demand from that same assumed distribution, run the policy against it like a real warehouse would, and count how often it actually stocked out — the closest thing to a controlled experiment you can run on a formula without waiting a literal year.

## Setup

You'll simulate **SKU 1 (Alpine Shell Jacket)** using the numbers you should already have from Exercises 1–2:

| Input | Value |
|---|---|
| Mean daily demand `d̄` | 4800/365 ≈ 13.15 |
| Daily demand std dev `σ_d` | 9.5 |
| Mean lead time `L` | 12 days |
| Lead-time std dev `σ_L` | 2.0 days |
| Order quantity `Q` (EOQ, rounded) | 234 |
| Reorder point `ROP` (95% CSL, `z`=1.645) | 227 |

## Tasks

### Part A — build the simulator

1. Write a day-by-day simulation for **365 days** with this logic each day:
   - Receive any order whose lead time has elapsed.
   - Draw today's demand from `Normal(d̄, σ_d)`, rounded to a non-negative integer (`max(0, round(...))`).
   - Fulfill demand from on-hand stock. If demand exceeds on-hand, the shortfall is **lost** (not backordered) — record it as unmet demand and set on-hand to 0.
   - Compute inventory position (on-hand + any order already in transit). If it's at or below `ROP` **and no order is currently outstanding**, place a new order for `Q` units, arriving after a random lead time drawn from `Normal(L, σ_L)` (rounded, minimum 1 day).
   - Track whether **any** stockout occurred since the last order was placed — that flags the cycle just completed as a "stockout cycle" for the cycle-service-level count.

2. Use a **fixed random seed** so your run is reproducible, and report which seed you used.

```python
import random

def simulate(seed, d_mean, d_std, L_mean, L_std, Q, ROP, days=365):
    random.seed(seed)
    on_hand = ROP + Q // 2          # arbitrary but reasonable starting inventory
    pending = []                     # list of (arrival_day, qty)
    total_demand = 0
    total_unmet = 0
    had_stockout_this_cycle = False
    cycle_stockout = []              # one entry per completed reorder cycle

    for day in range(1, days + 1):
        arrivals = [q for (a, q) in pending if a == day]
        for q in arrivals:
            on_hand += q
        pending = [(a, q) for (a, q) in pending if a != day]

        demand = max(0, round(random.gauss(d_mean, d_std)))
        total_demand += demand
        if demand > on_hand:
            total_unmet += demand - on_hand
            on_hand = 0
            had_stockout_this_cycle = True
        else:
            on_hand -= demand

        inv_position = on_hand + sum(q for (a, q) in pending)
        if inv_position <= ROP and not pending:
            lead = max(1, round(random.gauss(L_mean, L_std)))
            pending.append((day + lead, Q))
            cycle_stockout.append(had_stockout_this_cycle)
            had_stockout_this_cycle = False

    cycle_stockout.append(had_stockout_this_cycle)  # count the trailing partial cycle
    realized_csl = 1 - sum(cycle_stockout) / len(cycle_stockout)
    fill_rate = 1 - total_unmet / total_demand
    return realized_csl, fill_rate, len(cycle_stockout)
```

*(This uses the standard-library `random` module deliberately, so results are exactly reproducible from the seed alone — if you rewrite it with `numpy.random`, use `numpy.random.seed()` instead and expect slightly different but similarly-shaped numbers, since the two generators don't produce identical sequences from the "same" seed.)*

### Part B — run it and report

3. Run `simulate(seed=42, d_mean=4800/365, d_std=9.5, L_mean=12, L_std=2.0, Q=234, ROP=227)`. Report the realized cycle-service level, the fill rate, and the number of completed reorder cycles.

4. **Compare to the design target.** The policy was sized for a 95% CSL. Is your single-year realized CSL exactly 95%? Should it be? *(Think about how many reorder cycles occur in one year — Lecture 1 said ~20.5 for this SKU. With that few cycles, how much sampling noise would you expect around a 95% target?)*

5. **Run it 50–200 times with different seeds** and average the realized CSL across runs. This averages out the single-year noise from Task 4. Report the mean realized CSL across your runs, and its standard deviation across runs.

### Part C — connect to Lecture 2's fill-rate discussion

6. Compare your simulated fill rate to your simulated cycle-service level. Which is higher? Does that match Lecture 2 section 6's claim that fill rate tends to read higher than CSL for the same policy? Explain why, using your own simulation's numbers (how big was the typical shortfall relative to `Q`?).

## Expected results (reference run)

Using the exact code above with `seed=42`: **22 orders placed**, **23 completed cycles** (including the trailing partial cycle), **2 stockout cycles**, realized CSL **≈ 91.3%**, fill rate **≈ 99.4%** (total demand ≈5,154 units, total unmet ≈29 units). That single run reads meaningfully below the 95% target — expected, given only ~20 cycles/year. Averaged across 200 independent seeds, the mean realized CSL converges to **≈ 95.4%** (standard deviation across runs ≈ 4.7 percentage points, single-run range roughly 82%–100%) — confirming the formula is correct **on average**, even though any one year can land well off target purely from sampling noise. Fill rate stays consistently high (mean ≈ 99.6% across the 200 runs) — confirming Lecture 2's point that fill rate reads higher than CSL because typical shortfalls are small relative to `Q` = 234.

## Done when…

- [ ] Your single-seed run is within a few points of the reference numbers above (exact values will differ slightly with different random-number implementations, but the shape — CSL noticeably below target on some seeds, fill rate consistently high — should hold).
- [ ] Your multi-seed average CSL lands close to 95% (within a percentage point or two).
- [ ] You can explain, in your own words, why a single year of data is not enough to validate a service-level target, and what would be enough.

## Stretch

- Change `ROP` to the 99% target from Exercise 2 (`z`=2.33) and rerun the multi-seed average. Confirm the realized CSL rises accordingly, and report the extra average on-hand inventory (in units and in dollars, using `unit_cost * holding_pct`) that the higher target cost you across the simulated year.
- Modify the simulator to **backorder** unmet demand instead of losing it (carry a negative on-hand balance, fill it from the next arrival first). How does that change your interpretation of "stockout" and your realized fill-rate calculation?

## Submission

Commit `exercise-03.py` (and a short `exercise-03-notes.md` with your reported numbers) to your portfolio under `c40-week-05/exercise-03/`.
