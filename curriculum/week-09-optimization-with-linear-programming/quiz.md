# Week 9 — Quiz

Fifteen questions. Lectures closed. Aim for 12/15 before starting Week 10. A mix of multiple-choice and short "what would you write" questions — the answer key explains the *why*, not just the letter.

---

**Q1.** In a linear program, which of these is **not** allowed in the objective function or a constraint?

- A) $3x_1 + 2x_2$
- B) $x_1 - 5x_2 + 10$
- C) $x_1 \cdot x_2$
- D) $-x_1 + 4x_2 \le 100$

<details>
<summary>Answer</summary>

**C** — $x_1 \cdot x_2$ multiplies two decision variables together, which is nonlinear. Everything else (sums, constants, single-variable terms) stays linear.

</details>

---

**Q2.** The fundamental theorem of linear programming says that if an optimal solution exists, it can always be found at:

- A) The center of the feasible region
- B) A corner (vertex) of the feasible region
- C) Any point where all constraints have slack
- D) The point closest to the origin

<details>
<summary>Answer</summary>

**B** — the fundamental theorem of LP: if an optimum exists, at least one occurs at a vertex (corner) of the feasible region. This is why simplex only ever needs to check corners.

</details>

---

**Q3.** In the jackets/backpacks production-mix example (Lecture 1), both the fabric and stitching-hour constraints were binding at the optimum. What does "binding" mean?

- A) The constraint is violated
- B) The constraint holds with equality — the resource is fully used
- C) The variable's value is zero
- D) The constraint was removed from the model

<details>
<summary>Answer</summary>

**B** — binding means the constraint holds with equality at the optimum; the resource is fully consumed, with zero slack.

</details>

---

**Q4.** `scipy.optimize.linprog` only **minimizes**. To solve a maximization problem with it, you should:

- A) Set a `maximize=True` flag
- B) Negate the objective coefficients, minimize, then negate the result back
- C) It's impossible — use PuLP instead
- D) Reverse all the constraint inequalities

<details>
<summary>Answer</summary>

**B** — negate the coefficients, run `linprog` (which minimizes), then negate the returned objective value back to get the true maximum.

</details>

---

**Q5.** A transportation problem has 4 plants and 6 distribution centers. How many decision variables does the base model have (before adding any dummy source/sink)?

- A) 10
- B) 24
- C) 4
- D) 6

<details>
<summary>Answer</summary>

**B** — 24. A transportation problem has one variable per (plant, DC) pair: 4 × 6 = 24.

</details>

---

**Q6.** In the Week 9 seed network's transportation problem, why is the supply constraint written as `≤` and the demand constraint as `≥`, rather than both as `=`?

- A) `=` constraints are not supported by PuLP
- B) It's a stylistic choice with no real effect
- C) `≤`/`≥` still finds the same optimum when supply and demand totals match, but stays feasible (and meaningful) if the numbers in the source tables ever drift out of balance
- D) `≥` constraints solve faster than `=` constraints

<details>
<summary>Answer</summary>

**C** — writing supply as `≤` and demand as `≥` still finds the same optimum when totals match exactly, but stays feasible and meaningful (rather than immediately infeasible) if a seed table's numbers ever drift so supply and demand aren't perfectly balanced.

</details>

---

**Q7.** In the solved Week 9 network, all three plants (El Paso, Guadalajara, Ho Chi Minh) are fully used — zero slack on every supply constraint. What can you conclude about their shadow prices?

- A) All three must have the same nonzero shadow price
- B) All three must have a shadow price of zero, since "binding" means "no room to improve"
- C) They can differ — a constraint being binding does not guarantee its shadow price is nonzero, and in this network one plant's shadow price is exactly zero despite being fully used
- D) Shadow prices only exist for demand constraints, never supply constraints

<details>
<summary>Answer</summary>

**C** — binding (zero slack) does not guarantee a nonzero shadow price. In the Week 9 network, all three supply constraints bind, but Ho Chi Minh's shadow price comes out to exactly zero while Guadalajara's and El Paso's do not.

</details>

---

**Q8.** A shadow price of \$0 on a fully-used (binding) supply constraint tells you:

- A) That plant's capacity is worthless and could be eliminated entirely with no consequence
- B) A small **increase** in that capacity wouldn't reduce cost further — it says nothing reliable about what a **decrease** would do
- C) The plant is not actually part of the optimal solution
- D) The constraint was formulated incorrectly

<details>
<summary>Answer</summary>

**B** — a zero shadow price on a binding constraint is a statement about small *increases* only (no further benefit); it says nothing about what a *decrease* — especially a large one — would cost, since that can shift which constraints bind entirely.

</details>

---

**Q9.** What is the key difference between a linear program (LP) and a mixed-integer program (MIP)?

- A) A MIP's objective must be nonlinear
- B) An LP can only have one constraint
- C) A MIP requires at least one variable to take only integer (often binary 0/1) values, while an LP allows every variable to take any real value in its range
- D) MIPs can never be solved with PuLP

<details>
<summary>Answer</summary>

**C** — the defining feature of a MIP is that at least one variable is restricted to integers (commonly binary 0/1), unlike an LP where every variable can take any real value within its bounds.

</details>

---

**Q10.** In the facility-location model, the capacity constraint is written as $\sum_r x_{kr} \le \text{capacity}_k \cdot y_k$ rather than a separate `if y_k == 0 then no shipping` rule. Why does multiplying capacity by the binary $y_k$ work?

- A) It doesn't — this is a common but incorrect pattern
- B) When $y_k = 0$, the right side becomes 0, forcing all shipments out of that site to zero automatically; when $y_k = 1$, it becomes the ordinary capacity limit
- C) PuLP requires all constraints to include at least one binary variable
- D) It converts the constraint into an equality

<details>
<summary>Answer</summary>

**B** — multiplying capacity by the binary variable is exactly what makes the constraint behave like an on/off switch: zero when closed, the normal limit when open, with no extra logic required.

</details>

---

**Q11.** The "LP relaxation" of a MIP is:

- A) A completely different, unrelated model
- B) The same model with every integer/binary variable's integrality requirement dropped, allowing fractional values
- C) A simplified model with fewer constraints
- D) Only used for transportation problems, never facility location

<details>
<summary>Answer</summary>

**B** — the LP relaxation is the identical model with integrality dropped, letting binary/integer variables take fractional values — useful as a diagnostic and as the first step of branch and bound.

</details>

---

**Q12.** In the Lecture 3 facility-location result, the solver **closed** Austin East despite it being the incumbent DC. What was the deciding factor?

- A) Austin East had the lowest shipping costs on every lane
- B) Austin East's fixed cost was the highest of the four candidates, and that cost wasn't earned back by a shipping-cost advantage large enough to justify it
- C) Austin East's capacity was too small to matter
- D) The model was biased against incumbent facilities by design

<details>
<summary>Answer</summary>

**B** — Austin East's fixed cost ($80,000, the highest of the four) wasn't offset by a big enough shipping-cost advantage over the alternatives (especially the new Phoenix candidate), so the solver found it cheaper overall to close it.

</details>

---

**Q13.** A solver reports a model's status as `"Unbounded"`. What does this almost always mean?

- A) The problem has too many variables
- B) A constraint is missing — the objective can improve forever without hitting any real-world limit
- C) The problem was solved correctly and this is a valid final answer
- D) `Unbounded` and `Infeasible` mean the same thing

<details>
<summary>Answer</summary>

**B** — "Unbounded" means the objective can improve without limit because some real-world constraint was left out of the model; it is never a valid real-world answer, since no real resource is infinite.

</details>

---

**Q14.** Total plant supply in a transportation network is 12,000 units; total DC demand is 14,000 units. What is the standard way to keep the model solvable rather than simply infeasible?

- A) Reduce every DC's demand proportionally until it matches supply
- B) Add a dummy/emergency source with capacity equal to the 2,000-unit shortfall and a high cost on its lanes, so the solver can show exactly where and how much demand would go unmet
- C) Remove the demand constraints entirely
- D) It cannot be solved under any circumstances

<details>
<summary>Answer</summary>

**B** — the standard fix is a dummy source: give it capacity equal to the shortfall and a deliberately high cost, so the solver still finds a feasible, informative answer instead of just reporting "infeasible" with no further detail.

</details>

---

**Q15.** In the multi-period capacitated production plan (Challenge 2), the inventory balance constraint for quarter $t$ is written as an **equality** ($I_t = I_{t-1} + p_t - \text{demand}_t$), not an inequality. Why does it need to be an equality?

- A) PuLP does not support inequality constraints
- B) An inequality would let inventory be created or destroyed without being accounted for by actual production or demand, breaking the physical accounting the model is supposed to represent
- C) Equalities always solve faster than inequalities
- D) It doesn't need to be — an inequality would work identically

<details>
<summary>Answer</summary>

**B** — an inequality would let the model treat inventory as if it could materialize or vanish without being tied to actual production and demand, which would let the solver "cheat" the physical accounting the constraint exists to enforce.

</details>

**Scoring:** 12+ → start Week 10. 9–11 → re-read the lecture sections behind your misses, especially the shadow-price questions (7, 8) if you got those wrong — that distinction trips up almost everyone the first time. <9 → re-read all three lectures from the top before moving on; this week's concepts compound directly into Week 10's S&OP reconciliation.

---
