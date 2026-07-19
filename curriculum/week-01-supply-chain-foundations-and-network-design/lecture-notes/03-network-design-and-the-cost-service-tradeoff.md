# Lecture 3 — Network Design & the Cost-Service Trade-off

> **Duration:** ~2 hours. **Outcome:** You can reason about how many echelons a network needs, explain centralized vs. distributed facility strategy with the square root law, describe cross-docking and facility roles, and use a cost-service framework to argue *why* a network should look one way rather than another — with numbers, not instinct.

Lecture 1 gave you the pieces of a chain; Lecture 2 gave you the numbers that tell you whether it's healthy. This lecture is about the decision that shapes both: **where do the facilities go, and how many of them should there be?** Get this wrong and no amount of good forecasting or lean warehousing fixes it — you're optimizing the wrong shape.

## 1. Echelon count is a design choice

Lecture 1 showed Crunch Gear's jacket crossing five echelons: mill → hardware supplier → factory → DC → retailer/customer. Every one of those echelons was a *choice*, not a physical necessity. Consider the two extremes for the *finished-goods* half of the chain (factory to customer):

**Direct-ship (zero extra echelons):** the factory ships every order straight to the end customer or retailer. No DC in between.
- *Pro:* no warehousing cost, no double-handling, no inventory sitting idle in a DC.
- *Con:* every order pays full international/long-haul freight, lead times are long and variable (weeks, not days), and there's no place to consolidate small orders into cheaper truckload shipments.

**Multi-echelon (factory → national DC → regional DC → customer):** the standard structure most retailers and consumer brands use.
- *Pro:* bulk freight from factory to DC is cheap per unit; the DC can hold safety stock close to demand, so customer-facing lead time shrinks to days; regional DCs let different SKUs consolidate into full truckloads for that region.
- *Con:* each added echelon adds handling cost, adds a holding point where capital sits idle, and adds a leg where something can go wrong (loss, damage, a missed cross-dock window).

**The general rule:** add an echelon when the freight and lead-time savings it produces are worth more than the extra handling and inventory-holding cost it adds. That's not a slogan — Section 4 turns it into an actual number.

## 2. Centralized vs. distributed — the core trade-off

Once you've decided you need at least one DC layer, the next question is: **one big DC, or several smaller regional ones?**

| | **Centralized** (1 large DC) | **Distributed** (several regional DCs) |
|---|---|---|
| **Total safety stock needed** | Lower — demand variability pools across all customers into one location (Section 3) | Higher — each DC needs its own buffer against its own local demand swings |
| **Facility fixed cost** | Lower — one lease, one set of racking, one WMS instance, one management team | Higher — N leases, N sets of fixed costs |
| **Inbound freight (factory → DC)** | Cheaper per unit — full truckloads/containers to one point | Same total volume, but often split into smaller, less-efficient inbound shipments |
| **Outbound freight & delivery lead time** | Worse — every customer, near or far, ships from the one location | Better — customers near a regional DC get short, cheap, fast delivery |
| **Resilience to a single-site disruption** | Worse — one fire, strike, or flood takes out 100% of capacity | Better — losing one regional DC still leaves the others serving their zones |

Neither column wins in general. A company selling heavy, low-margin, slow-moving industrial parts to a handful of big customers usually centralizes (freight and holding cost dominate). A company selling to consumers nationwide with next-day delivery expectations usually distributes (speed to customer dominates, and the up-charge for guaranteed fast shipping justifies the extra facility cost). Most real networks land somewhere in between — a "hybrid" with one national DC for slow movers and several forward regional nodes stocking the fast-moving, popular SKUs.

## 3. The square root law of inventory — why centralizing pools risk

Here's *why* centralizing reduces total safety stock, with the actual (simplified) relationship. If a company splits its safety stock across `n` identical, independent locations instead of holding it all in one, the *total* safety stock needed scales roughly with `√n`, not `n`:

```
Total safety stock (n locations) ≈ Total safety stock (1 location) × √n
```

**Worked intuition.** Suppose Crunch Gear needs $400,000 of safety stock to buffer demand variability at a single national DC. If it instead splits into 4 regional DCs, each covering a quarter of the country:

```
√4 = 2
Total safety stock across 4 DCs ≈ $400,000 × 2 = $800,000
```

Twice the safety stock investment to serve the *same* total demand, purely from splitting one pool of variability into four independent, smaller pools. **Why does this happen?** In one big pool, an unusually strong week in the Southeast is often offset by an unusually weak week in the Northwest — the variability partially cancels out ("pools") before it ever has to be buffered. Split into four separate DCs, the Southeast DC has to buffer its own strong week on its own, with no offsetting weak week elsewhere in *its* pool to lean on. Distributing loses that natural cancellation, and the cost shows up as extra safety stock, warehouse by warehouse.

This is the single most important reason distributed networks cost more in inventory — and it's exactly why the decision in Section 2 is a genuine trade-off (faster delivery, more capital tied up) rather than one obviously correct answer.

## 4. Facility roles

Not every box on the map is a plain warehouse. Common roles, and why the distinction matters:

- **Plant** — makes the product. Fixed location, driven by labor cost, raw material access, and trade agreements (this is why so much apparel manufacturing sits in Southeast Asia).
- **DC / RDC (regional distribution center)** — holds inventory and fulfills orders. The workhorse of Section 2's centralize/distribute decision.
- **Cross-dock facility** — receives inbound freight and reloads it onto outbound trucks the **same day**, with little or no put-away into storage. Used when you want the consolidation benefit of a hub (combine many small inbound shipments into efficient outbound loads) *without* paying for inventory to sit there. Grocery and retail replenishment networks rely heavily on cross-docking to move perishable or fast-moving goods without the cost of a full storage DC.
- **Fulfillment center (FC)** — a DC purpose-built for high-volume single-unit picking (e-commerce/DTC orders), as opposed to a traditional DC optimized for full-case or full-pallet picking to retail stores. Crunch Gear's DTC orders and its wholesale-retailer orders often ship from architecturally different facilities even when they're in the same building, because picking one jacket for a website order and picking 500 jackets for a retail chain are almost completely different physical operations.

Getting the *role* wrong — running DTC single-unit picking out of a facility racked and staffed for full-pallet wholesale — is one of the most common and expensive network-design mistakes as a company grows a new channel inside an old network.

## 5. The cost-service trade-off curve

Every network decision in this lecture — echelon count, centralize vs. distribute, which facility role to build — ultimately trades off against **total landed cost vs. service level**. The relationship isn't a straight line; it curves sharply upward at the high end:

```
Cost
 ^                                                    ●  99.9%
 |                                              ●
 |                                        ●  99%
 |                              ●     
 |                     ●    95%
 |            ●
 |    ●    90%
 |
 +---------------------------------------------------------> Service level (OTIF / fill rate)
```

Going from 90% to 95% service is usually cheap — a modest safety-stock increase, maybe a slightly faster (but not radically more expensive) freight lane. Going from 99% to 99.9% is disproportionately expensive — it typically requires expedited freight as a standing policy, forward stock at every location, and enough safety stock to cover rare, extreme demand spikes almost all the time. **This is why "just ship everything faster and stock more of everything" is not a strategy** — it's an unbounded cost curve. The actual skill is choosing *where on that curve* each product/customer segment belongs: a company's top 20% of SKUs (or its highest-value customers) might justify living up near 99%, while slow-moving tail SKUs are fine at 90–92%, because the cost of stocking them richly everywhere isn't worth what it buys.

## 6. Total landed cost — the framework that ties it together

When comparing two network designs (this week's Exercise 3 and Challenge 2 both ask you to do exactly this), compare them on **total landed cost**, not any single line item in isolation:

```
Total landed cost = Inventory holding cost
                   + Transportation cost (inbound + outbound)
                   + Facility fixed cost (lease, labor, systems)
                   + Cost of service failures (lost sales, expedite fees, customer churn from stockouts)
```

A network that looks cheaper on freight alone but forces more safety stock (Section 3) or racks up stockout losses (a real, if harder-to-measure, cost) can easily be more expensive overall. **Worked comparison**, Crunch Gear choosing between 1 national DC and 4 regional DCs, annualized:

| Cost component | 1 centralized DC | 4 regional DCs |
|---|---:|---:|
| Safety stock carrying cost (25%/yr of $400k vs. $800k, from Section 3) | $100,000 | $200,000 |
| Facility fixed cost | $450,000 | $4 × $150,000 = $600,000 |
| Outbound delivery cost & speed | Higher cost per order, 4–6 day transit | Lower cost per order, 1–2 day transit |
| Estimated stockout/expedite cost | $180,000 (slower replenishment to distant regions) | $60,000 (closer stock, faster reaction) |
| **Total (excl. delivery, shown separately)** | **$730,000** | **$860,000** |

On this cut, centralized is **$130,000/year cheaper** — but the distributed network delivers roughly 3–4 days faster to most customers. Whether that's worth $130,000/year depends entirely on whether faster delivery drives enough incremental sales or retention to justify it — a question this lecture can frame but can't answer for you without real conversion/retention data. That's precisely the judgment call Challenge 2 asks you to make and defend this week.

## 7. Check yourself

- Give one reason to add an echelon to a network, and one reason to remove one.
- State the square root law in your own words. If Crunch Gear splits from 1 DC to 9 regional DCs, what multiplier does total safety stock scale by?
- What's the difference between a cross-dock facility and a traditional DC? Why would a company deliberately choose *not* to store inventory at a hub?
- Sketch the shape of the cost-service curve. Why is going from 99% to 99.9% service disproportionately expensive compared to 90% to 95%?
- List the four components of total landed cost. Which one is hardest to measure, and why does that make it easy to accidentally ignore?
- In the worked comparison (Section 6), what piece of missing data would let you decide, with confidence, whether the distributed network is worth its extra $130,000/year?

That's the full toolkit for Week 1. The mini-project asks you to put all three lectures together: map a real chain, and compute and interpret its KPIs from a supplied dataset.

## Further reading

- **Eppen, G.D. (1979), "Effects of Centralization on Expected Costs in a Multi-Location Newsboy Problem"**, *Management Science* — the formal derivation behind the square root law of inventory pooling: <https://pubsonline.informs.org/doi/10.1287/mnsc.25.5.498>
- **ASCM Supply Chain Dictionary — "Cross-Docking," "Distribution Center," "Network Design":** <https://www.ascm.org/learning-development/certifications-credentials/scmdictionary/>
- **MIT CTL (Center for Transportation & Logistics) — "Supply Chain Network Design" overview materials:** <https://ctl.mit.edu/research> — browse for network design and facility location working papers, several of which are free.
- **Chopra, S. & Meindl, P., *Supply Chain Management: Strategy, Planning, and Operation*** — the standard graduate textbook; most university libraries carry it, and its network-design chapters go well beyond this lecture if you want the full mathematical treatment.
