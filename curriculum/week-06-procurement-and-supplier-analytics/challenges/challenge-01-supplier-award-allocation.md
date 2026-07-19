# Challenge 1 — Supplier Award Allocation

**Time:** ~1.5 hours. **Builds on:** Lecture 2 (scorecards), Lecture 3 (TCO), Exercise 2.

## The situation

Crunch Gear's Outerwear division needs **6,000 CMT-sewn jackets** for Q2 2026 — up from the roughly 4,500 the division ordered from CMT suppliers in Q1. Three suppliers are qualified or qualifiable to sew Outerwear-grade jackets, each with a hard quarterly **capacity ceiling** their factory has confirmed they cannot exceed:

| Supplier | Quarterly capacity | TCO per unit (Lecture 3) | Track record |
|---|---:|---:|---|
| Andes Stitch Works | 5,000 units | $24.088 | 3 Outerwear POs in Q1; highest price, best quality (0.92% defect), worst OTD (33.3%) |
| Pacific Rim Garments | 4,200 units | $23.007 | 3 Outerwear POs in Q1; mid price, worst quality (3.51% defect), worst OTD (33.3%) |
| Northstar Apparel Mfg | 2,000 units | $16.337 | **2 Accessories POs in Q1 only** — has never sewn an Outerwear-grade jacket for Crunch Gear; would need a first-article qualification run before any real Outerwear volume |

Total available capacity across all three (11,200 units) comfortably exceeds the 6,000 needed — this is not a supply-shortage problem. It's an **allocation** problem: how much of the 6,000 should go to each supplier, and why?

## Your task

Produce an award allocation (units per supplier, summing to exactly 6,000, respecting every capacity ceiling) and a written justification. Specifically:

1. **Start with the naive TCO-minimizing allocation** — fill the cheapest supplier's capacity first, then the next cheapest, until you reach 6,000. State what that allocation is and its blended (volume-weighted) TCO per unit.
2. **Now argue for a different allocation** that you'd actually recommend, and defend it. At minimum, address:
   - **Concentration risk.** What happens to Outerwear production if your single largest awarded supplier has a bad month — a quality escape, a labor stoppage, a shipping delay? Is your recommended allocation resilient to that, or does it recreate the same single-point-of-failure risk Lecture 1 flagged for Alpine Weave Mills?
   - **Northstar's qualification gap.** Northstar's scorecard and TCO both look excellent — but on *two Accessories orders*, not Outerwear. Would you award them real Outerwear volume immediately, a small pilot batch, or hold off a quarter? Justify the number you land on with more than "to be safe."
   - **The price you're paying for risk mitigation.** If your recommended allocation isn't the TCO-minimizing one, compute exactly how much more (in total dollars, at 6,000 units) it costs relative to the naive allocation from Step 1. A recommendation that can't state its own cost isn't complete.
3. **State one condition that would change your recommendation** — e.g., "if Northstar completes a successful 500-unit Outerwear pilot with <1.5% defects, I'd increase their Q3 allocation to X."

## Deliverable

A short memo (`award-allocation.md`, 300–500 words) with:

- A table: supplier, awarded units, unit TCO, subtotal cost.
- The blended TCO per unit for your recommended allocation vs. the naive TCO-minimizing allocation, and the dollar gap between them.
- Your written justification covering all three numbered points above.

## What a strong answer covers

- The naive allocation is computed correctly and its blended TCO is stated (this is a straightforward "sort by TCO, fill capacity" calculation — get it right before arguing for something else).
- The recommended allocation respects every capacity ceiling and sums to exactly 6,000.
- Concentration risk is discussed with a number attached (e.g., "no single supplier exceeds X% of total awarded volume") — not just asserted in prose.
- Northstar's qualification gap is treated as a real constraint, not ignored because the TCO number is attractive. A strong answer typically caps Northstar's *initial* award well below their full 2,000-unit capacity, precisely because two orders isn't enough evidence to bet a third of Outerwear production on.
- The dollar cost of the risk-mitigated allocation vs. the naive one is computed, not hand-waved — this is the number that makes the recommendation defensible to a CFO who will otherwise just ask "why aren't we taking the cheapest option?"
- The "what would change my mind" condition is specific and measurable, not vague ("if they do well" is not specific; "if they complete a 500-unit pilot at <1.5% defect rate" is).
