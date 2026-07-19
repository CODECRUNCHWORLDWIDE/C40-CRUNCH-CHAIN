# Challenge 2 — Labor Capacity Planning for a Peak Day

**Goal:** Take a stated peak-day order-volume forecast, size a picking crew to it with a realistic utilization buffer, and quantify — in lines, orders, and minutes, not vaguely — exactly what happens if the plan is one picker short.

**Estimated time:** 1.5 hours.

This challenge is self-contained — it introduces its own peak-day numbers below, decoupled from Austin East DC's regular one-month pick profile. You don't need to re-query `pick_lines` for this one; you need Lecture 3 Section 5's labor-sizing formula and your own arithmetic.

## The scenario

Crunch Gear is running a **Founders Day Sale**, and Austin East DC's forecasting team (this course's Weeks 3–4 tools, applied to marketing's promo-volume estimate) projects:

| Input | Value |
|---|---|
| Forecast order lines for the peak Saturday | **5,000 lines** |
| Average lines per order | **2.9** |
| Target pick rate (batched picking, current slotting) | **125 lines/hour per picker** |
| Shift length | **8 hours**, with a 30-minute break (**7.5 productive hours**) |
| Target utilization (Lecture 3 Section 5's realistic-slack guidance) | **80%** of productive shift time |

## Tasks

1. **Convert the pick rate.** `125 lines/hour` → how many minutes per line? Confirm it matches `60/125 = 0.48 min/line`.

2. **Total picker-minutes required.** `5,000 lines × 0.48 min/line` = how many picker-minutes does the whole day's forecast need? Convert to picker-hours too.

3. **Usable minutes per picker, at target utilization.** A picker's shift gives 450 productive minutes (7.5 hours), but Lecture 3 Section 5 said never plan a schedule at 100% utilization. At the stated **80%** target, how many *usable* minutes does one picker actually contribute?

4. **Pickers needed.** Divide Task 2's total required minutes by Task 3's usable-minutes-per-picker, and **round up** (a fractional picker isn't schedulable — Lecture 3, Section 5). How many pickers does Austin East DC need to schedule for the peak Saturday?

5. **Understaffed by one.** Suppose only **one fewer picker** than Task 4's answer actually shows up (call-outs happen). Compute:
   - Total usable picker-minutes with the reduced crew.
   - The shortfall in minutes versus what Task 2 requires.
   - That shortfall converted back into **lines** (divide by 0.48 min/line) and then into **orders** (divide by 2.9 lines/order).
   - State the result as a sentence a shift lead could act on: "With one picker short, approximately ___ orders will not clear the floor by end of shift and will need overtime or next-day fulfillment."

6. **Sensitivity check.** Does raising the utilization target to a more aggressive **85%** let the understaffed crew (Task 5's headcount) clear the full forecast? Show the arithmetic either way.

7. **The other lever.** Instead of adding a picker, how much would the **pick rate** need to improve (fewer minutes per line) for the understaffed crew from Task 5 to clear the full 5,000-line forecast within their usable minutes? Express the answer both as a new `min/line` target and as a `lines/hour` target, and connect it back to this week's other two levers — is a ~10% pick-rate improvement more plausible from a *better slotting plan* (Lecture 2) or from *bigger batches* (Lecture 3)? Defend your answer with the size of the savings each lever produced earlier this week.

## Expected results (spot checks)

- Task 2: **2,400 picker-minutes** required (40 picker-hours).
- Task 4: **7 pickers** needed at 80% target utilization.
- Task 5 (6 pickers scheduled): a shortfall of **240 picker-minutes**, which is **500 lines**, which is roughly **172 orders** that miss the cutoff.
- Task 6: even at 85% utilization, 6 pickers still fall short — the gap shrinks but doesn't close.

## Why this matters

"How many people do I need today" is one of the most common questions an operations analyst answers, and it's almost always asked under time pressure, the day before a known peak. A defensible answer needs three things this challenge forces you to produce together: the *required* capacity (from a real forecast), a *realistic* usable-capacity number (not shift length taken at face value), and the *cost of being wrong* in either direction — understaffing (missed orders, overtime, angry customers) and overstaffing (idle labor cost). A number with no sensitivity check attached is not a staffing plan; it's a guess with decimal places.

## Submission

Commit `challenge-02.py` (or a worked `challenge-02.md` if you'd rather show the arithmetic by hand) to your portfolio under `c40-week-08/challenge-02/`.
