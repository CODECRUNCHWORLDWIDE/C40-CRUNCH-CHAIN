# Lecture 3 — Reorder Policies & the Newsvendor

> **Duration:** ~2 hours. **Outcome:** you can describe and size an (s,Q) policy, an (R,S) policy, and a base-stock policy, explain why periodic review needs more safety stock than continuous review, and solve a single-period newsvendor problem for a perishable or one-shot product.

Lectures 1 and 2 gave you the two numbers that drive most inventory decisions: **how much** to order (`EOQ`) and **when** to trigger that order (`ROP`, built from safety stock). This lecture packages those numbers into an actual operating **policy** — a rule someone (or some system) follows every day without re-deriving the math — and covers the one important case where the whole "reorder" framing doesn't apply at all: a product you only get to order **once**.

## 1. (s,Q) — continuous review, fixed order quantity

**The policy:** watch inventory position continuously (or close to it — daily, in most real systems). The moment inventory position drops to the reorder point `s` (what we've been calling `ROP`), place an order for a fixed quantity `Q` (the `EOQ`).

```
IF inventory_position <= s:
    place order for Q units
```

"Inventory position" — not just "on-hand" — matters here: it's on-hand stock **plus** anything already on order but not yet received, **minus** any backorders. If you triggered a reorder yesterday and check on-hand today, on-hand alone would look low and you'd double-order; inventory position accounts for the order already in flight.

**When you'd use it:** any environment with real-time or daily inventory visibility — a modern POS or WMS, which is most retail and DTC operations today (including Crunch Gear's warehouse system). It's the default policy in this course and the one this week's exercises and mini-project build.

**What it needs:** `s = ROP = d̄*L + z*σ_DLT` (Lecture 2) and `Q = EOQ = sqrt(2DS/H)` (Lecture 1). Notice the two numbers came from two completely different lectures, computed independently, and then combined into one operating rule — that's normal; EOQ answers "how much," safety stock answers "how much buffer," and (s,Q) is just the container that holds both.

## 2. (R,S) — periodic review, order-up-to level

**The policy:** review inventory position on a fixed schedule — every `R` days, not continuously — and each time, order enough to bring inventory position up to a target level `S` (the "order-up-to level," a different `S` from Lecture 1's ordering cost — unfortunately both are conventionally called `S` in the inventory literature; context tells them apart).

```
EVERY R days:
    order (S - inventory_position) units
```

**When you'd use it:** whenever continuous tracking isn't available or isn't practical — a supplier who only accepts orders on Tuesdays, a vendor-managed-inventory arrangement where a rep visits weekly, or multiple SKUs that get ordered together on one consolidated truck to hit a freight minimum, reviewed on the same calendar cadence rather than each SKU triggering independently.

**Why it needs more safety stock than (s,Q), for the same service level:** under (s,Q), the risk window is just the lead time `L` — from the moment you *notice* you need to reorder to the moment the order arrives, and you notice instantly. Under (R,S), you might have just missed a review — inventory could drop to a dangerously low level the day after a review and then sit there, unreplenished, until the *next* review `R` days later, plus the lead time `L` after that. So the exposure window is `R + L`, not just `L`:

```
S = d̄*(R + L) + z * σ_(R+L)

σ_(R+L) = sqrt( (R+L) * σ_d²  +  d̄² * σ_L² )
```

Same formula shape as Lecture 2's `σ_DLT`, just with `(R+L)` substituted for `L` everywhere demand-exposure duration appears.

### Worked comparison — Alpine Shell Jacket (SKU 1), (s,Q) vs. (R,S) with a 14-day review cycle

| | (s,Q), continuous review | (R,S), 14-day review |
|---|---|---|
| Exposure window | `L` = 12 days | `R+L` = 14+12 = 26 days |
| `σ` over the window | `sqrt(12*9.5² + 13.15²*2²) ≈ 42.1` | `sqrt(26*9.5² + 13.15²*2²) ≈ 55.1` |
| Safety stock at 95% CSL (`z`=1.645) | `1.645 * 42.1 ≈ 69.3` units | `1.645 * 55.1 ≈ 90.7` units |
| Trigger / target | `ROP ≈ 227` units | `S ≈ 433` units |

Reviewing only every 14 days instead of continuously costs Crunch Gear about **31% more safety stock** on this one SKU (90.7 vs 69.3 units), purely because of the longer exposure window — nothing about demand or lead time itself changed. **This is the real, quantifiable cost of infrequent review**, and it's exactly the number you'd put in front of an operations manager who's deciding whether it's worth the systems investment to move from weekly counts to real-time inventory tracking.

## 3. Base-stock policy — the (S-1, S) special case

**The policy:** a base-stock policy is (R,S) with `R` shrunk to the smallest possible unit — review (and potentially reorder) after **every single unit of demand**. Every time one unit sells, replace it with an order for exactly one unit, keeping inventory position at a constant target `S` at all times. It's sometimes written **(S-1, S)**: reorder point is one below the target, because the moment you're at `S-1` (one unit sold), you order one unit to get back to `S`.

**When you'd use it:** expensive, slow-moving items where holding even one extra unit of safety stock is costly, and where demand is infrequent enough that "review after every sale" is operationally realistic — think Crunch Gear's Summit Down Parka (SKU 2: $165/unit, only 1,500 units/year, ≈4/day) rather than Wool Beanies selling 8,400/year. Base-stock policies are also the standard building block for multi-echelon inventory models (Challenge 2 this week), because they compose cleanly across stages of a supply chain.

## 4. Choosing a policy — a quick decision guide

| Situation | Policy |
|---|---|
| Real-time or daily inventory visibility, independent SKU ordering | **(s,Q)** |
| Fixed review calendar (weekly truck, VMI visit, supplier order day) | **(R,S)** |
| Expensive, slow-moving, low-volume item | **Base-stock (S-1,S)** |
| Multiple SKUs consolidated onto one periodic order (freight minimum, joint replenishment) | **(R,S)**, reviewed together |
| A single, one-time, perishable, or use-it-or-lose-it decision | **Newsvendor** (next section — not a "reorder" policy at all) |

## 5. The newsvendor problem — when there is no next cycle

Everything so far assumed a **repeating** cycle: order, sell down, reorder, forever. Some decisions don't get that luxury. A limited-edition product drop, a seasonal item ordered once before the season, produce with a shelf life shorter than the reorder cycle, an event T-shirt — you place **one order**, demand happens **once**, and whatever you didn't sell is a loss (markdown, donation, disposal) while whatever demand you didn't cover is a **permanently** lost sale, not a backorder you'll fill next week. This is the **newsvendor problem** (named for a news-stand vendor deciding how many copies of tomorrow's paper to buy, knowing unsold copies are worthless the next day).

### The trade-off: underage vs. overage cost

Two costs, and they pull in opposite directions:

- **`Cu` (underage cost)** — the cost of ordering **one unit too few**: the lost margin on a sale you can't make (and possibly goodwill on top, though we'll keep it simple and use margin).
- **`Co` (overage cost)** — the cost of ordering **one unit too many**: what you lose on a unit that doesn't sell — the difference between what it cost you and whatever salvage value you recover (a markdown rack, a liquidator, a donation write-off).

```
Cu = selling_price - unit_cost
Co = unit_cost - salvage_value
```

### The critical ratio and the optimal quantity

The optimal order quantity balances the *marginal* risk of the next unit: order it if the expected benefit of having it (in case demand is high) outweighs the expected cost of it going unsold (in case demand is low). Working through that balance (the standard newsvendor derivation, omitted here — see Further Reading if you want the full proof) gives a strikingly clean result. Define the **critical ratio**:

```
CR = Cu / (Cu + Co)
```

The optimal order quantity `Q*` is the quantity such that the probability of demand being **at or below** `Q*` equals `CR` — in other words, `Q*` is the `CR`-th **quantile** of the demand distribution:

```
Q* = F⁻¹(CR)
```

If demand is approximately normal with mean `μ` and standard deviation `σ`, this becomes as simple as the safety-stock formula from Lecture 2:

```
Q* = μ + z*σ,   where z = Φ⁻¹(CR)
```

**Read the critical ratio for intuition:** if `Cu` (cost of running out) is much bigger than `Co` (cost of overstock), `CR` is close to 1, `z` is large, and you order well above the mean — better to have leftovers than miss sales. If `Co` dominates, `CR` is small, `z` is negative, and you deliberately order *below* the mean, accepting some lost sales to avoid a pile of unsold, unsalvageable stock.

### Worked example — a limited seasonal drop

Crunch Gear is producing a one-time run of a **collaboration trail jacket** for a single fall drop — no reorder is possible before the design cycles out.

| Input | Value |
|---|---|
| Selling price | $150 |
| Unit cost | $70 |
| Salvage value (end-of-season markdown) | $40 |
| Forecast demand | Normal, `μ` = 1,800 units, `σ` = 400 units |

**Step 1 — underage and overage cost:**

```
Cu = 150 - 70 = $80   (margin lost per unit of unmet demand)
Co = 70 - 40  = $30   (loss per unit left unsold, after markdown)
```

**Step 2 — critical ratio:**

```
CR = 80 / (80 + 30) = 80/110 ≈ 0.727
```

**Step 3 — z-score for that ratio, and optimal order:**

```
z = Φ⁻¹(0.727) ≈ 0.604
Q* = 1,800 + 0.604 * 400 ≈ 1,800 + 241.6 ≈ 2,042 units
```

**Reading it:** because it's more expensive to run out (`$80`) than to be stuck with extras (`$30`), the optimal order sits **above** the mean forecast (2,042 vs. 1,800) — Crunch Gear should deliberately over-order relative to its point forecast, by design, because the cost structure of *this specific product* rewards erring toward too much rather than too little. Change the salvage value to $10 instead of $40 (harder to unload leftovers) and `Co` rises to $60, `CR` falls to 0.571, `z` falls to about 0.18, and `Q*` drops to just above the mean (~1,872) — the model responds correctly to a worse "stuck with it" outcome by ordering closer to, not above, the forecast.

```python
from scipy.stats import norm

def newsvendor_q(price, cost, salvage, mu, sigma):
    cu = price - cost
    co = cost - salvage
    cr = cu / (cu + co)
    z = norm.ppf(cr)
    return mu + z * sigma, cr, z

q_star, cr, z = newsvendor_q(price=150, cost=70, salvage=40, mu=1800, sigma=400)
print(f"CR={cr:.3f}  z={z:.3f}  Q*={q_star:.0f}")
```

### Newsvendor and safety stock are the same idea, wearing different clothes

Compare `Q* = μ + z*σ` (newsvendor) to `ROP = d̄*L + z*σ_DLT` (Lecture 2's safety stock). Same structure: a central estimate, plus a z-scaled standard deviation that leans the decision toward whichever error costs more. The difference is entirely about what happens **after**: a normal reorder cycle gives you another chance next cycle to correct for being wrong, so the z-target is about a *service level* you can sustain repeatedly. The newsvendor has **no next cycle** — you get exactly one shot, so the z-target is derived directly from the dollar cost of each type of error, not from an assumed target percentage. Whenever you're staring at a one-shot, perishable, or use-it-or-lose-it stocking decision, reach for the newsvendor's critical ratio instead of a service-level table.

## 6. Check yourself

- What's the difference between the exposure window for (s,Q) and for (R,S)? Which is bigger, and what specifically does the extra length come from?
- Why does a base-stock policy suit an expensive, slow-moving SKU better than a fast-moving, cheap one?
- Write the critical ratio formula. If `Cu` is ten times `Co`, is `CR` closer to 0 or 1? What does that imply about where `Q*` sits relative to the mean forecast?
- A newsvendor item has `Cu = Co`. What is `CR`, what is `z`, and where does `Q*` land relative to the mean? (This is a useful sanity-check case to memorize.)
- Why can't you apply a standard 95%-service-level safety-stock formula to a one-time seasonal product the same way you'd apply it to a fast-moving SKU with weekly reorders?

## Further reading

- **APICS/ASCM — Order review policies (s,Q / R,S / base-stock):** <https://www.ascm.org/>
- **SciPy — `scipy.stats.norm.ppf` (inverse CDF / quantile function):** <https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.norm.html>
- **Wikipedia — Newsvendor model (derivation of the critical ratio):** <https://en.wikipedia.org/wiki/Newsvendor_model>
