# Lecture 1 — Warehouse Operations & Flows

> **Duration:** ~2 hours. **Outcome:** You can draw the receive-to-ship flow from memory, name what happens (and what can go wrong) at each step, and explain — in dollars and minutes, not vague terms — where a distribution center's labor cost actually accumulates.

A distribution center (DC) looks, from the outside, like a big box that inventory goes into and orders come out of. Inside, it's six distinct jobs, done by different people, in a fixed order, and every one of them has its own failure modes and its own cost driver. Learn the six steps once and you can walk into *any* warehouse — Crunch Gear's or anyone else's — and immediately know which question to ask first.

## 1. The receive-to-ship flow

```
RECEIVING → PUTAWAY → STORAGE → PICKING → PACKING → SHIPPING
```

Six steps, one direction. Product enters at receiving and exits at shipping; everything in between is the warehouse "earning its keep" by making picking fast and cheap when the order finally comes in.

| Step | What happens | Who touches it | Primary cost driver |
|------|--------------|-----------------|----------------------|
| **Receiving** | Inbound trucks are unloaded, cartons are counted, checked against the purchase order, and staged | Receiving dock crew | Dock-door time, count accuracy |
| **Putaway** | Received product is moved from the dock to its assigned storage location | Forklift/pallet-jack operators | Travel distance dock → slot |
| **Storage** | Product sits in its slot until an order calls for it | Nobody — this is a *cost*, not a task | Cubic footage occupied × time held |
| **Picking** | A picker walks to a slot, pulls the quantity an order needs, and carries it toward packing | Order pickers | **Travel distance + touch time** (this week's focus) |
| **Packing** | Picked items for one order are boxed, cushioned, and labeled | Packers | Box selection, labor per order |
| **Shipping** | Packed orders are staged by carrier, loaded, and manifested | Shipping dock crew | Dock-door time, carrier cutoff windows |

Week 7 covered what happens *after* shipping — modes, routing, carriers. This week lives entirely inside the box, from the moment a pallet comes off Week 7's inbound truck to the moment a packed order rolls onto an outbound one.

## 2. Where the labor dollars actually go

Ask any warehouse operations manager which step eats the most labor hours and the answer is almost always the same: **picking.** Not receiving, not packing — picking. Here's why, with real industry-reported splits (your mileage varies by DC, but the *shape* is consistent everywhere):

| Activity | Typical share of total warehouse labor hours |
|----------|----------------------------------------------|
| Picking | **~50%** |
| Receiving & putaway | ~15% |
| Packing | ~20% |
| Shipping | ~10% |
| Other (cycle counting, replenishment, etc.) | ~5% |

Picking dominates for a structural reason: receiving and shipping happen **once per shipment** (a truckload of 500 cases is one receiving event), but picking happens **once per order line** — Crunch Gear's Austin East DC processes 76 orders and 225 order lines in the one-month sample you seeded this week, which means 225 separate walk-to-a-slot-and-pull events versus a handful of inbound truck receipts. Multiply a small per-event cost by hundreds of events and it becomes the dominant number on the labor budget. **That's why this entire week is about picking, and specifically about the two levers that control picking cost: where things are stored (slotting) and how trips are grouped (batching).**

Of the time a picker spends on a single pick, further break it down:

| Component | Share of pick time | Controlled by |
|-----------|---------------------|----------------|
| **Travel** — walking to and from the slot | **50–70%** | Slotting (Lecture 2) and batching (Lecture 3) |
| **Search** — finding the exact bin/shelf | 10–15% | Slot labeling, location discipline |
| **Pick + verify** — grab, count, scan | 15–20% | Pick technology (scanner vs. paper), item packaging |
| **Documentation/travel to pack station** | 5–10% | Workflow design |

**Travel is the single biggest component of pick time in almost every conventional (person-to-goods) warehouse**, which is exactly why slotting — literally just *where you put things* — is the highest-leverage, lowest-capital-cost improvement an analyst can propose. You don't need to buy a conveyor or hire more people to cut travel time; you need to move product to better locations, which costs a Tuesday afternoon and a forklift.

## 3. Storage strategies: fixed vs. random vs. velocity-based

How a DC decides *where* a SKU lives is called its **storage strategy**, and there are three common ones:

**Fixed-slot storage.** Every SKU has one assigned, permanent home. Simple to learn, easy to audit ("Trail Socks are always in aisle 3"), but wastes space — an out-of-stock SKU's slot sits empty instead of being used for something else — and, if assigned carelessly, produces exactly the kind of mismatch you'll find in this week's seed data.

**Random (dynamic) slot storage.** Any unit can go in any open slot; a warehouse management system (WMS) tracks where everything actually is and tells pickers where to go. Maximizes space utilization (no slot sits empty just because "that's Product X's spot") but requires a real-time system — you can't run random storage on a clipboard.

**Velocity-based (ABC) storage.** A hybrid: fast-moving SKUs get fixed, close-in "golden zone" locations; slow-moving SKUs live further out, sometimes in random/bulk storage. This is the industry-standard approach for person-to-goods picking, and it's what Lecture 2 teaches you to design. Austin East DC currently runs something that *looks* like fixed-slot storage but was actually assigned **alphabetically by SKU name** — which is fixed storage with a random-with-extra-steps outcome, because a SKU's name has nothing to do with how often it's ordered.

## 4. The golden zone

Within velocity-based storage, the **golden zone** (also called the "power zone" or "hot zone") is the set of locations closest to the pack/ship station — the shortest possible round trip. In a person-to-goods warehouse like Austin East DC, that's a small number of prime shelf locations right off the pack line. Everything about slotting strategy comes down to one question: **which SKUs earn a spot in that scarce, valuable real estate?**

The answer is never "whichever SKU we received first" or "alphabetical" — it's **whichever SKUs get picked the most often**, because every trip to that slot is a trip you're paying for, and a slot 20 feet away costs a fraction of what a slot 175 feet away costs, multiplied by however many times a month someone has to walk there. Lecture 2 turns "picked the most often" into a precise, computable ranking: **ABC analysis.**

## 5. Reading a warehouse layout

This week's `warehouse_slots` table (from the [week setup](../README.md)) encodes a simplified single-line layout: a pack/ship station at distance 0, with slots fanning out in three zones.

```sql
SELECT zone, COUNT(*) AS n_slots, MIN(distance_ft) AS closest_ft, MAX(distance_ft) AS farthest_ft
FROM warehouse_slots
GROUP BY zone
ORDER BY MIN(distance_ft);
```

```
 zone    | n_slots | closest_ft | farthest_ft
---------+---------+------------+-------------
 Golden  |       6 |         20 |          40
 Middle  |      10 |         55 |         110
 Reserve |       8 |        130 |         235
```

Real warehouses are two-dimensional — aisles, bays, levels — and distance is usually computed along **rectilinear travel paths** (you walk down aisles, you don't cut diagonally through racking), not straight lines. This week deliberately uses a **single travel-distance number per slot** (already computed as the round-trip-equivalent walking distance from the pack station) so you can focus on the *slotting decision* — which SKU goes in which zone — without also having to solve a two-dimensional routing problem. Week 7's vehicle-routing lecture is where you built genuine two-dimensional path-finding; this week reuses that instinct at DC scale, one dimension simplified on purpose.

## 6. Putaway: the flip side of picking

Every unit that gets picked first had to get **put away** — carried from the receiving dock to its storage slot. This means a slotting decision has a cost on *both* ends: a golden-zone slot is cheap to pick from but, because it's small, needs **more frequent replenishment** (someone has to keep restocking it from reserve as it empties). A reserve-zone slot is expensive to pick from but can hold a whole pallet, so it needs restocking only rarely.

This is the tension every slotting plan has to resolve: **put your fastest movers closest to shipping (cheap picks), but check that a small golden-zone slot doesn't need refilling so often that the replenishment trips eat the pick-travel savings.** Lecture 2 gives you the velocity ranking; Challenge 1 makes you check this exact trade-off against Austin East DC's real case-pack sizes.

## 7. What "good" looks like at the end of this week

By Sunday, you will have:

1. Classified all 20 Austin East SKUs into A/B/C velocity tiers from the real 225-line pick profile.
2. Computed, in feet and in minutes, exactly what the current (alphabetical) slotting costs versus a velocity-based re-slot.
3. Batched a day of real orders into a wave and measured how many fewer trips that produces.
4. Sized a picking crew to a stated throughput target and reported the three numbers a fulfillment manager tracks every day: **throughput** (orders or lines per hour), **order cycle time** (release to ship), and **pick productivity** (lines or units per labor-hour).

None of that requires new racking, new software, or a bigger crew. It requires reading the data that's already sitting in the warehouse management system and asking, one query at a time, "is this actually the cheapest way to run this building?" Lecture 2 starts answering that question with the first real number: the ABC velocity ranking.
