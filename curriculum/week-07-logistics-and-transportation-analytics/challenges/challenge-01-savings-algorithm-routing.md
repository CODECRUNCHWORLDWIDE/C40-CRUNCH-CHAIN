# Challenge 1 — Savings-Algorithm Routing

**Time:** ~90 minutes. **Difficulty:** Medium-hard. **There's a right implementation, but room for style.**

## The scenario

Your capacitated nearest-neighbor solution from Exercise 2 works — 4 trucks, 345.7 miles — but Lecture 2 promised something better was possible. Your job is to implement the **Clarke-Wright savings algorithm** from scratch, in Python, against the same `delivery_stops` network and the same 120-case truck capacity, and prove — with numbers — that it beats nearest-neighbor.

## Background: the algorithm, precisely

1. **Start** with every customer stop on its own dedicated round-trip route from the depot: `[0, i, 0]` for every stop `i`.
2. **Compute the savings** for every pair of stops `(i, j)`:
   ```
   savings(i, j) = dist(0, i) + dist(0, j) - dist(i, j)
   ```
   This is how many miles you'd save by visiting `i` and `j` back-to-back on one truck instead of on two separate round trips.
3. **Sort** all pairs by savings, descending.
4. **Walk the sorted list.** For each pair `(i, j)`, merge their two routes into one **if and only if**:
   - `i` and `j` are currently on *different* routes, **and**
   - `i` is at an **endpoint** of its route and `j` is at an **endpoint** of its route (you can only join routes at their open ends — you can't splice into the middle of an existing route), **and**
   - the combined load of the merged route does not exceed the 120-case capacity.
5. **Stop** when you've walked every pair. Whatever routes remain are your final solution.

## Your task

1. **Implement it.** Write `savings_algorithm(stop_ids, demand, dist_fn, capacity)` returning a list of routes (each a list of `stop_id`s from depot to depot), following the steps above exactly. Reuse your `dist()` function from Exercise 2.

2. **Validate it.** Before trusting your total distance, check:
   - Every one of the 12 customer stops appears in **exactly one** route.
   - No route's total demand exceeds 120.
   - Every route starts and ends at stop `0`.

   Write these three checks as actual assertions in your code, not just eyeballing the output.

3. **Compare it.** Compute total distance for your savings-algorithm solution and put it side by side with your Exercise 2 capacitated nearest-neighbor result:

   | Method | Trucks | Total distance |
   |---|---:|---:|
   | Capacitated nearest-neighbor (Exercise 2) | 4 | 345.7 mi |
   | Clarke-Wright savings (this challenge) | ? | ? |

   State the percentage improvement: `(nn_distance - savings_distance) / nn_distance * 100`.

4. **Explain one merge decision.** Pick one pair `(i, j)` that your algorithm merged, and one pair with high savings that it *rejected* (because of the endpoint rule or the capacity rule). Explain in one sentence each why the algorithm made that call.

## Constraints

- Do this in **Python**, not SQL — the merge logic is inherently iterative and state-tracking, which SQL is a poor fit for.
- You may use `pandas`/`numpy` for the distance matrix, but the merge loop should be plain Python you wrote yourself — not an off-the-shelf VRP solver. The point is understanding the mechanics.
- Don't hand-tune your result by eyeballing the map and rearranging stops after the fact — the algorithm's output is the answer, warts and all.

## Hints

<details>
<summary>On the "endpoint" rule</summary>

Track each route as an ordered list. A stop `i` is at an endpoint if it's `route[1]` (right after the depot) or `route[-2]` (right before the depot). When you merge route `A` ending in `i` with route `B` starting with `j`, the new route is `A[:-1] + B[1:]` (drop `A`'s trailing depot, drop `B`'s leading depot, splice). Watch your direction — you may need to reverse one of the routes if the savings pair matches on the "wrong" ends.

</details>

<details>
<summary>On why savings usually wins</summary>

Nearest-neighbor only ever asks "what's closest to where I am right now" — it has no concept of the depot except as a start/end point. Savings explicitly reasons about **how much extra distance the depot round-trip costs** for each pair, which is precisely the quantity that determines whether two stops belong on the same truck. It's a more informed heuristic because it's using more of the problem's structure.

</details>

## How success is judged

| Signal | Weak submission | Strong submission |
|---|---|---|
| Correctness | Routes overlap stops, or a route exceeds capacity | All 3 validation assertions pass, provably |
| Comparison | "It's better" with no number | Exact mile figures and a computed percentage improvement |
| Understanding | Can't explain any specific merge decision | Explains one accepted and one rejected merge, correctly, in terms of the algorithm's rules |
| Code quality | One giant unstructured script | Clear `savings_algorithm()` function, reusable, with assertions |

## Submission

Commit `challenge-01.py` (with your written comparison and explanation as comments or a companion `challenge-01.md`) to your portfolio under `c40-week-07/challenge-01/`.
