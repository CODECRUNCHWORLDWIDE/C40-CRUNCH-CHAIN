# Challenge 2 — Build a Should-Cost Model

**Time:** ~1.5 hours. **Builds on:** Lecture 3 — Total Cost of Ownership & Sourcing Strategy.

## What a should-cost model is

A **should-cost model** flips the usual procurement question. Instead of asking "what does this supplier charge?" it asks "what *should* this cost to produce, built up from labor, overhead, materials, and a reasonable margin?" — independent of what any specific supplier happens to quote. The gap between the should-cost estimate and the actual price paid is your **negotiation opportunity**: a concrete, defensible number to bring into a supplier conversation, instead of "your price feels high."

Recall from Lecture 3 that Crunch Gear's **CMT** (Cut-Make-Trim) suppliers are paid only for the *labor and finishing* to sew a jacket — Crunch Gear supplies the fabric separately (that's why `Fabric` and `CMT` are separate spend categories in this week's dataset). So the should-cost model you build here estimates a **CMT fee**, not a fully landed garment cost.

## Task 1 — Build the bottom-up estimate

Using the cost structure below (typical for a mid-tier apparel CMT operation), build the should-cost estimate in SQL, Python, or worked by hand with each step shown:

| Line item | Assumption |
|---|---|
| Direct labor | 1.2 hours per jacket at $8.50/hour |
| Factory overhead | 40% of direct labor (utilities, equipment depreciation, floor supervision) |
| SG&A | 10% of (labor + overhead) |
| Profit margin | 10% of (labor + overhead + SG&A) |

```python
labor_hours = 1.2
labor_rate  = 8.50

labor    = labor_hours * labor_rate
overhead = 0.40 * labor
subtotal_1 = labor + overhead
sga      = 0.10 * subtotal_1
subtotal_2 = subtotal_1 + sga
margin   = 0.10 * subtotal_2
should_cost = subtotal_2 + margin

print(f"Labor:     ${labor:.4f}")
print(f"Overhead:  ${overhead:.4f}")
print(f"SG&A:      ${sga:.4f}")
print(f"Margin:    ${margin:.4f}")
print(f"Should-cost: ${should_cost:.4f}")
```

**Expected:** labor $10.20, overhead $4.08, SG&A $1.428, margin $1.5708, **should-cost ≈ $17.28/unit**.

## Task 2 — Compare should-cost to actual price paid

Pull the volume-weighted average actual price paid to Andes Stitch Works and Pacific Rim Garments (Exercise 2 / Lecture 2 already computed these) and compute the gap in dollars and as a percentage of should-cost.

**Expected:**

| Supplier | Actual price paid | Should-cost | Gap ($) | Gap (% over should-cost) |
|---|---:|---:|---:|---:|
| Andes Stitch Works | $22.462 | $17.279 | $5.183 | 30.0% |
| Pacific Rim Garments | $20.867 | $17.279 | $3.588 | 20.8% |

Both suppliers are priced meaningfully above the should-cost estimate — Andes more so. This is a **normal, expected finding**, not evidence of a scandal: should-cost models are deliberately conservative (they assume efficient, typical operations, and real factories carry costs a spreadsheet buildup won't capture — rework, seasonal idle capacity, smaller batch sizes than the model assumes). A 15–25% gap is a realistic, usable negotiation range in apparel CMT sourcing; a 200%+ gap would be a sign your assumptions are wrong, not that you found a windfall.

## Task 3 — Stress-test your own assumptions

A should-cost model is only as good as its inputs, and every input above was an assumption, not a fact. Pick **two** assumptions and show how much the should-cost estimate — and therefore the negotiation gap — moves if you're wrong about them:

1. What if direct labor actually takes 1.5 hours, not 1.2 (a more complex jacket style)?
2. What if the local labor rate is $10.00/hour, not $8.50 (a higher-cost sourcing region)?
3. What if factory overhead runs 55% of labor instead of 40% (an older, less efficient facility)?

Recompute should-cost under at least two of these and show the new gap vs. Andes and Pacific Rim's actual prices. State, in one sentence per scenario, whether the negotiation opportunity you found in Task 2 survives the stress test.

## Task 4 — Tie back to Lecture 3's TCO risk question

Lecture 3, Section 5 found that Andes's TCO premium over Pacific Rim would only flip in Pacific Rim's disfavor at an unrealistically high customer-defect-escape cost (breakeven ≈ $157/unit). Now that you have a should-cost floor for *both* suppliers, answer: is Pacific Rim's price closer to should-cost (in percentage terms) than Andes's? What does that suggest about which supplier has *more room* to negotiate versus which one is already pricing closer to a defensible cost basis?

## What a strong answer covers

- Task 1's arithmetic is correct and every intermediate step (labor → overhead → SG&A → margin) is shown, not just the final number — a should-cost model's value is in the auditable buildup, not the headline figure.
- Task 2 correctly identifies the gap is a *negotiation range*, not a claim that the supplier is overcharging or acting in bad faith.
- Task 3 actually recomputes the model under at least two alternate assumptions (not just discusses them in prose) and reports how much the "opportunity" shrinks or grows.
- Task 4 connects this challenge's should-cost floor back to Lecture 3's TCO analysis, and draws a specific, numeric conclusion about which supplier has more negotiating room — not just "they're both a bit high."
