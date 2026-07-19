# Exercise 1 — Map a Real Chain End to End

**Goal:** Take the vocabulary from Lecture 1 (nodes, echelons, the three flows) and apply it to a real product — not the Crunch Gear example, a genuinely different one — so the concepts stop being "things in the lecture" and start being a lens you can point at anything.

**Estimated time:** 1 hour.

## Setup

Pick **one** real product you can plausibly trace. Good choices: a bag of coffee, a smartphone, a pair of sneakers, a carton of milk, a piece of furniture, a book. Avoid picking something you can't reason about at all (e.g., a highly classified defense product) — you need to be able to make *reasonable, stated* assumptions about nodes you can't directly verify.

Create a file `map.md` for your answers.

## Tasks

### 1. List the nodes (list, minimum 6)

For your chosen product, list every node type from raw material to end customer. Use Lecture 1's vocabulary (raw material supplier, component supplier, plant/manufacturer, DC/warehouse, retailer, DTC customer, reverse logistics) — but adapt names to your product ("dairy farm" instead of "raw material supplier," if that's clearer). For any node you're not 100% sure exists, write it anyway and mark it `(assumed)`.

### 2. Draw the echelons (diagram, ASCII is fine)

Draw your nodes as a left-to-right chain of echelons, connected by lanes, in the style of Lecture 1's diagram:

```
[Node] --> [Node] --> [Node] --> [Node] --> [Node]
```

Label at least one lane with its likely transportation mode (ocean freight, truck, rail, air, last-mile parcel/delivery).

### 3. Trace the three flows (short written answers)

For your product, answer each in 2–3 sentences:

- **Product flow:** describe the physical journey in your own words, start to finish.
- **Information flow:** name one piece of information that flows *downstream → upstream* (e.g., a sales/consumption signal) and one that flows *upstream → downstream* (e.g., a shipment notice or backorder alert).
- **Cash flow:** describe, in general terms, who pays whom and roughly when. You don't need real numbers — reason about *sequence* (who's paid first, who waits longest to get paid).

### 4. Identify the decoupling point

Where in your chain does the process switch from **push** (forecast-driven, before a real order exists) to **pull** (order-driven, triggered by an actual sale)? State which node it's at and why you placed it there.

### 5. One bullwhip scenario

Invent one small, believable disruption or demand spike for your product (a viral social-media moment, a holiday spike, a weather event affecting a farm). In 3–4 sentences, describe how you'd expect that signal to distort as it moves upstream through your chain's echelons — which node is likely to overreact, and why.

## Expected result (self-check)

- At least 6 distinct nodes, correctly ordered from raw material to consumer.
- A diagram with labeled lanes and at least one named transportation mode.
- All three flows addressed separately — many first attempts collapse information and cash flow into one answer; make sure yours doesn't.
- A decoupling point identified with a one-sentence justification, not just a location.

## Done when…

- [ ] `map.md` has all 5 sections filled in for a product other than Crunch Gear.
- [ ] Every node has a label; assumed nodes are marked `(assumed)`.
- [ ] You can explain, out loud, why the decoupling point sits where you put it.
- [ ] The bullwhip scenario names a specific node and a specific reason it overreacts — not just "it gets worse upstream."

## Stretch

- Add a second, alternate decoupling point your chosen company *could* choose instead, and one sentence on what trade-off moving it would create (faster delivery vs. more inventory risk, per Lecture 1 §6).
- Estimate — even roughly — how many echelons your product's chain has versus Crunch Gear's five, and whether you'd expect more or fewer given the product's shelf life, value density, and customization needs.

## Submission

Commit `map.md` to your portfolio under `c40-week-01/exercise-01/`.
