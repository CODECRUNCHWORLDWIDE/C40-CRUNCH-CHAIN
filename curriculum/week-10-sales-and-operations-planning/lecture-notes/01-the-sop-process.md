# Lecture 1 — The S&OP Process

> **Duration:** ~2 hours. **Outcome:** You can name the four stages of the monthly S&OP cycle, explain what each stage produces and who owns it, run the demand-review query that surfaces where Sales and Statistics disagree, and explain — with a real number — why "one set of numbers" is the entire point of the process.

## 1. The problem S&OP exists to solve

Every function in a company keeps its own number for "how much are we going to sell/make/spend/earn next month":

- **Sales** has a pipeline and a gut feel, updated constantly, usually optimistic.
- **Demand planning** has a statistical forecast, built from history, usually more conservative and slower to react to news.
- **Operations** has a capacity plan built months ago, and it doesn't move just because Sales got excited about a deal.
- **Finance** has a revenue and margin target baked into the annual operating plan (AOP), locked in during last quarter's budget cycle.

Left alone, these four numbers **drift apart every single month**, because each function updates on its own schedule from its own inputs, and nobody is forcing them back into alignment. The drift is invisible until it isn't: Sales books more than Ops can make, or Ops builds more than Sales can sell, or Finance's targets quietly stop matching reality on both sides. By the time someone notices — a stockout, a markdown, a missed quarter — the gap has been compounding for months.

**Sales & Operations Planning (S&OP)** is the standing monthly process that forces those four numbers back into one number, every month, before the gap gets expensive. It isn't a meeting about *whether* to plan — everyone already has a plan. It's a meeting about *reconciling the plans that already exist* into one that the whole company can execute against.

## 2. The four stages, in order

```
 1. DEMAND REVIEW  →  2. SUPPLY REVIEW  →  3. RECONCILIATION  →  4. EXECUTIVE SIGN-OFF
 (Sales + Demand      (Ops: can we meet     (Ops + Finance +      (Leadership: approve
  Planning agree        that demand?         Sales negotiate       the ONE plan, or send
  on one forecast)       what's the gap?)     the trade-offs)       it back for rework)
```

Each stage has a distinct owner, a distinct input, and a distinct output. Skipping a stage — going straight from "Sales wants X" to "Ops, make X happen" — is exactly how the drift in Section 1 gets built into the plan instead of caught by it.

```mermaid
flowchart LR
  A["Demand review"] --> B["Supply review"]
  B --> C["Reconciliation"]
  C --> D["Executive sign-off"]
  D -->|Rework requested| A
```

*The monthly S&OP cycle — four stages in order, with leadership able to send the plan back for rework instead of approving it.*

### Stage 1 — Demand review

**Owner:** Demand Planning, with Sales/Commercial as a required input, not a decision-maker who overrides on their own. **Input:** a statistical forecast (built the way Weeks 3–4 of this course taught — baselines, seasonality, trended history) *and* a sales/field forecast for the same period. **Output:** one **consensus forecast** per product family per month — the number the rest of the cycle runs on.

Look at Trail Footwear's April row in this week's `demand_plan` table:

```sql
SELECT month, stat_forecast_units, sales_input_units, consensus_forecast_units,
       sales_input_units - stat_forecast_units AS gap
FROM demand_plan
WHERE product_family = 'Trail Footwear'
ORDER BY month;
```

```
 month      | stat_forecast_units | sales_input_units | consensus_forecast_units | gap
------------+----------------------+--------------------+---------------------------+------
 2025-01-01 |                 8000 |               8200 |                      8200 |  200
 2025-02-01 |                 8400 |               8600 |                      8600 |  200
 2025-03-01 |                10000 |              10400 |                     10400 |  400
 2025-04-01 |                12200 |              13400 |                     12800 | 1200
 2025-05-01 |                13800 |              14200 |                     14200 |  400
 2025-06-01 |                12800 |              13000 |                     13000 |  200
```

Most months, the two forecasts are close (a 200–400 unit gap, roughly 2–4%) and the consensus just takes the sales number — small gaps aren't worth fighting over. **April is different.** The gap jumps to 1,200 units (nearly 10%) because Sales is banking on a new retail-partner launch the statistical model has zero history for — it can't see a deal that hasn't happened yet. Demand Planning, for its part, has no evidence the launch will hit its full run rate in month one. Neither side is *wrong*. The consensus of 12,800 is a **negotiated position**, not an average computed by a spreadsheet formula, and a well-run demand review writes down *why*: "12,800 assumes the retail launch delivers 50% of its target run rate in April, ramping to 100% by June — revisit if the launch date slips."

That written assumption is the single most valuable artifact of the demand review. Six weeks from now, when April's actuals come in, the team doesn't just see "we missed the forecast" — they see "we missed *this specific assumption*," which tells them exactly what to fix for May.

### The trap: consensus is not "always average the two"

A tempting shortcut is `consensus = (stat + sales) / 2` every time, computed with no judgment involved. Don't do this. Sometimes the statistical forecast is right and Sales is chasing one loud customer; sometimes Sales has real information (a signed contract, a canceled account) the statistical model can't see yet because it hasn't happened in history. The *process* — a documented conversation with a stated reason — is the point, not the arithmetic. A consensus number with no stated reason behind it is exactly as fragile as no consensus at all.

## 3. Stage 2 — Supply review

**Owner:** Operations/Supply Planning. **Input:** the consensus forecast from Stage 1, plus current capacity — regular production, available overtime, available subcontract/outsourced capacity, and current inventory position. **Output:** for every family and month, a clear answer to *"can we make this many, and if not, by how much are we short?"*

This is a mechanical check before it's a negotiation: take capacity, take the consensus forecast, take beginning inventory, and see if the ending position ever drops below the safety-stock floor. Here's the shape of the query, checked against **Backpacks & Bags** using only regular capacity (no overtime or subcontract yet — those are held in reserve as levers, not baked into the baseline plan):

```sql
SELECT d.month, d.product_family, d.consensus_forecast_units,
       s.regular_capacity_units,
       s.regular_capacity_units - d.consensus_forecast_units AS month_gap
FROM demand_plan d
JOIN supply_plan s USING (month, product_family)
WHERE d.product_family = 'Backpacks & Bags'
ORDER BY d.month;
```

```
 month      | product_family    | consensus_forecast_units | regular_capacity_units | month_gap
------------+-------------------+---------------------------+-------------------------+-----------
 2025-01-01 | Backpacks & Bags  |                      5100 |                    5500 |       400
 2025-02-01 | Backpacks & Bags  |                      5400 |                    5500 |       100
 2025-03-01 | Backpacks & Bags  |                      6800 |                    7000 |       200
 2025-04-01 | Backpacks & Bags  |                      8900 |                    8500 |      -400
 2025-05-01 | Backpacks & Bags  |                     10200 |                   10500 |       300
 2025-06-01 | Backpacks & Bags  |                      9600 |                   10000 |       400
```

April shows a **-400 raw gap** for Backpacks & Bags — regular capacity alone can't cover April's consensus forecast that month. That single negative number is the entire output of the supply review for this family: "we can cover five of six months from regular capacity alone; April needs help, and here's exactly how much." Whether that "help" comes from overtime, subcontract, pre-built inventory, or a request back to Demand to reconsider the forecast is *not* decided here — that's Stage 3. The supply review's job is to surface the gap precisely, not to solve it.

You'll build the full running-inventory version of this table — which accounts for beginning inventory carrying forward month to month, not just each month in isolation — in [Exercise 1](../exercises/exercise-01-build-a-supply-demand-table.md).

## 4. Stage 3 — Reconciliation

**Owner:** a cross-functional group — usually Ops, Finance, and a Demand/Sales representative, sometimes chaired by a "S&OP owner" role that reports to the COO or supply chain VP. **Input:** every gap the supply review surfaced. **Output:** a decision, on every gap, among a short list of real options:

| Option | What it costs | When it makes sense |
|---|---|---|
| **Add overtime/subcontract capacity** | A cost premium per unit (this week's `unit_cost_overtime` / `unit_cost_subcontract`) | Gap is moderate, capacity exists, and the extra cost is smaller than the value of the lost sale |
| **Pre-build inventory in an earlier surplus month** | Holding cost on the extra units, held earlier than needed | An earlier month has slack capacity and the product doesn't perish/obsolete quickly |
| **Accept a controlled shortage / backorder** | Lost or delayed revenue, customer goodwill | The gap is small, temporary, or the cost to close it exceeds the value of closing it |
| **Push back on the demand number** | None directly — but only valid if the forecast itself is genuinely shaky | The gap traces to an unproven assumption (like April's retail launch) rather than real, committed demand |

Notice what's *not* on this list: silently letting Ops build to their own capacity number and hoping Sales doesn't notice, or silently letting Sales keep selling against a number Ops never agreed it could hit. Both of those are what happens **without** S&OP — the entire value of Stage 3 is making the trade-off explicit and choosing it on purpose, with a number attached, instead of discovering it by accident in week four of the month.

## 5. Stage 4 — Executive sign-off

**Owner:** the leadership team (often the GM, COO, or CEO for the business unit). **Input:** the reconciled plan from Stage 3, with every gap's resolution attached and costed. **Output:** approval — or, just as often, a directive to go back and re-cut the plan under a different constraint ("that overtime spend is too high, go find $80K of it back another way").

This stage matters even when it feels like a rubber stamp, for one reason: **it is the moment the plan becomes the company's plan**, not Ops's plan or Sales's plan. After sign-off, a stockout in a constrained month isn't "Ops failed to hit Sales's number" — it's "the plan the company approved accepted that risk on purpose, for a stated reason." That distinction is worth protecting; it's what keeps the next month's demand review honest instead of adversarial.

## 6. A worked reconciliation decision

Stage 3's options table (Section 4) is abstract until you attach it to a real gap. Take Backpacks & Bags' April shortfall from Section 3: regular capacity alone is 400 units short of the 8,900-unit consensus forecast. Here's what a reconciliation conversation actually walking through the four options looks like, with numbers attached to each:

| Option | What it would cost this specific gap | Verdict |
|---|---|---|
| Overtime (April `overtime_capacity_units` = 850 at $23.10/unit) | 400 units × $23.10 = **$9,240** incremental | Comfortably covers the gap; overtime capacity (850) is more than double what's needed |
| Subcontract (April `subcontract_capacity_units` = 600 at $26.00/unit) | 400 units × $26.00 = **$10,400** incremental | Also covers it, but costs more per unit than overtime — no reason to pick this first |
| Pre-build in March (March has 200 units of raw surplus per Section 3's table) | Holding cost on 200 units for one extra month, ~$2.25/unit × 200 = **$450** | Cheapest option in isolation, but only closes half the 400-unit gap — March's own surplus isn't quite big enough alone |
| Accept the shortfall | $0 direct cost, but ~400 units × $95.00 price = **$38,000** of at-risk revenue if the sale is lost outright (not just delayed) | Only defensible if 400 units is genuinely disposable demand, which a 400-unit gap against an 8,900-unit month usually isn't |

Laid out this way, the decision is easy: **use April's own overtime.** It's the cheapest lever that fully closes the gap on its own, and at $9,240 against a $38,000 revenue-at-risk figure if the shortfall were simply accepted, the trade is not close. This is exactly the kind of table Stage 3 produces for *every* gap the supply review surfaces — not a gut call, a short comparison with real numbers on both sides.

## 7. Common failure modes — what happens when a stage gets skipped

Every one of these is a real, common way S&OP breaks down in practice. Recognizing them is as useful as knowing the process itself.

- **The "shadow forecast."** Sales keeps booking against their own number even after the demand review agreed to a lower consensus, because nobody enforces that the consensus is what actually gets used downstream. The demand review becomes theater — a meeting that happens, produces a number, and is then ignored.
- **Sandbagging.** A function deliberately understates its forecast or overstates its capacity constraint to build in a safety margin nobody else can see, because it's been burned before by an overly aggressive plan. This defeats the entire purpose of reconciliation — the "one set of numbers" is no longer honest, just differently wrong.
- **No owner for the gap.** A shortfall gets identified in the supply review, discussed in reconciliation, and then... nobody is explicitly assigned to execute the fix (schedule the overtime, place the subcontract order) by a specific date. The gap resurfaces, unresolved, next month.
- **Sign-off without teeth.** Leadership approves the plan, but nothing changes if a function quietly deviates from it mid-month — no different, functionally, than never having sign-off at all. The "company's plan" language in Section 5 only means something if deviating from the signed-off plan has a real cost (at minimum, having to explain why at the next cycle).
- **Treating S&OP as a one-time event instead of a cadence.** A great reconciliation this month means nothing if the same four stages don't run again next month, on schedule, whether or not last month's plan held up. The value compounds from catching drift *every* month, not from occasionally doing a heroic one-off reconciliation.

## 8. Why "one set of numbers" is the whole point

The phrase you'll hear in every real S&OP meeting is **"one set of numbers."** It means: after sign-off, Sales, Ops, and Finance are all working from the *same* consensus forecast, the *same* capacity plan, and the *same* revenue target — not three separate versions that each function quietly adjusted to make their own dashboard look better. A company running S&OP well can answer, instantly, "what are we planning to sell, make, and earn in April" with **one number per question**, agreed by everyone, traceable to a decision. A company not running it has three answers to each question and doesn't find out they disagree until the numbers actually collide in the real world.

That's the standard this whole week is building toward: every table you touch this week — demand, supply, inventory, finance — gets joined into that one reconciled view, in SQL, with every judgment call written down next to the number it produced.

## 9. Check yourself

- Name the four stages of the monthly S&OP cycle in order, and who owns each one.
- Why is `consensus = average(stat, sales)` the wrong default, even though it's the easiest formula to write?
- What's the difference between what the *demand review* produces and what the *supply review* produces?
- List the four options reconciliation has for closing a gap, and name one situation where each is the right call.
- In the Backpacks & Bags worked example (Section 6), why was overtime chosen over subcontract even though both fully closed the gap?
- Why does executive sign-off matter even when the plan mostly just gets rubber-stamped?
- Name one S&OP failure mode from Section 7 and explain, in one sentence, which stage of the cycle it undermines.
- Trail Footwear's April gap was 1,200 units between stat and sales forecasts. What is the actual *reason* given in this lecture for that gap — and why can't the statistical model see it on its own?

If those are automatic, Lecture 2 goes inside the supply review's toolbox: once you know a month is short, how do you actually decide whether to hire, run overtime, or hold inventory to cover it?

## Further reading

- **APICS/ASCM — Sales and Operations Planning body of knowledge overview:** <https://www.ascm.org/>
- **Council of Supply Chain Management Professionals — S&OP glossary entry:** <https://cscmp.org/CSCMP/Educate/SCM_Definitions_and_Glossary_of_Terms.aspx>
- **PostgreSQL — Window Functions (used throughout this week for running balances):** <https://www.postgresql.org/docs/current/tutorial-window.html>
