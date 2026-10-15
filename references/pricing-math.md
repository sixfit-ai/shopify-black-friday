# Pricing math

Every number the skill shows the merchant comes from these formulas. Use them
exactly. Show results in plain words ("you keep €21.40 per sale"), never the
formula itself, unless the merchant asks.

---

## 1. Inputs

| Symbol | Meaning | Source | If missing |
|---|---|---|---|
| `P` | Current selling price, as the customer sees it | Variant `price` | — |
| `v` | VAT rate included in `P` | `shop.taxesIncluded`. If true, ask the VAT rate per market (e.g. NL 21%, DE 19%, FR 20%). If false, `v = 0`. | Ask. Never assume 0 for an EU store. |
| `c` | Landed unit cost, ex VAT (purchase + freight + duties) | `inventoryItem.unitCost`, or the merchant's margin per category | Ask. No cost → no margin verdict. |
| `f` | Payment fee, % of the paid amount | Interview | 0, flagged "margins overstated" |
| `g` | Fixed payment fee per order | Interview | 0, flagged |
| `s` | Shipping subsidy per order: your outbound cost minus what the customer pays | Interview | 0, flagged |
| `n` | Average items per order | ShopifyQL `net_items_sold / orders` | 1 (conservative) |
| `r` | Return rate (share of units returned) | Product's own data (≥ 10 units), else category, else store, else interview | Proposed 30% (guide: apparel holiday returns approach ~30%), stated and agreed |
| `k` | Share of returned units you can resell at full value | Interview | 1, flagged "slightly overstated" |
| `h` | Cost per returned unit: return label you pay + processing | Interview | Always ask. No default. |
| `d` | Discount, as a fraction of `P` | Being decided | — |

## 2. Contribution per unit shipped

```
p    = P · (1 − d)                      price paid
N    = (1 − r) · p / (1 + v)            net revenue you keep, ex VAT, after refunds
COGS = c · (1 − r · k)                  goods you don't get back
FEES = f · p + g / n                    payment fees (assumed not refunded on returns)
SHIP = s / n                            shipping subsidy share
RET  = r · h                            return handling

C(d)  = N − COGS − FEES − SHIP − RET    contribution per unit shipped (€)
CM(d) = C(d) / N                        contribution margin, share of kept net revenue
```

`C(0)` and `CM(0)` are the full-price values. **Always strip VAT.** On a
VAT-inclusive store, skipping it overstates margin by `v / (1 + v)` of the
price, about 17 points at 21%.

## 3. Break-even volume uplift

How many more units a discount must sell to earn the same total contribution
as full price:

```
U(d) = C(0) / C(d) − 1            only if C(d) > 0; otherwise "loses money on every sale"
```

Say it plainly: *"At 20% off you need to sell <U>% more of this item just to
make the same profit as at full price."* (Use the computed value.)

**Compare with evidence, never with hope.** If last year's BFCM data exists,
compute the observed uplift:

```
L = units in last year's BFCM window ÷ average units in same-length windows
    over the 4 weeks before it
```

Per product if it sold ≥ 10 units in the window, else per category, else
store-wide. Verdict:

| Condition | Say |
|---|---|
| `L ≥ 1 + U(d)` | "Last year's demand suggests this pays for itself." |
| `1 + U(d) > L ≥ 1 + U(d)/2` | "Borderline. It only pays if this year beats last year." |
| `L < 1 + U(d)/2`, or no data | "Unlikely to pay for itself on volume alone. Only worth it for a reason other than profit (traffic, new customers, clearing stock)." |

No history → say there's no evidence either way and show `U(d)` only. Never
invent an elasticity.

## 4. Tests per bucket

| Bucket | Passes at `d` when | Why |
|---|---|---|
| **Hero** | `CM(d) ≥ margin floor` and `C(d) > 0` | Hero deals must make money. |
| **Low-return boost** | Same as Hero | |
| **Clearance** | `C(d) ≥ clearance floor` (default **0**: every sale at least pays its own costs) | Dead stock's alternative isn't full price: it's a deeper January markdown, an outlet, or a write-off. Judge it against cash recovered, not full-price profit. The merchant can set a negative clearance floor to liquidate, and must say so explicitly. |

**Maximum safe discount** `d*`: the largest `d`, in 5-point steps up to the
bucket's cap, that passes its bucket test. Report it per product. It's the
single most useful number for the merchant.

**What to show per bucket:**
- **Hero / Low-return:** `d*`, € kept per sale at `d*` vs full price, and
  break-even uplift `U(d*)` with the last-year verdict.
- **Clearance:** `d*`, € recovered per sale (`C(d*)`), and the stock value
  moved if it all sells. **Don't show `U(d)` for Clearance.** Comparing to
  full-price profit is the wrong question for dead stock, and the number
  (often +1000%) only confuses.

**Problem products.** If `CM(0)` (full price) is already below the margin
floor, flag the product whatever its bucket: *"<Product> barely makes money
even at full price (<CM(0)>), mostly because of <the biggest cost: returns /
VAT / cost>. Discounting it loses money."* Suggest Exclude, or fixing the
cause (returns, cost, price) after BF.

## 5. Clean price points (amount off)

A percentage on €128 gives €102.40. If the merchant wants clean price points
(€99), use **amount off per item**:

```
amount     = P − target_price
d_equiv    = amount / P            run every test above with d_equiv
advertised = floor(d_equiv × 100)%  never round up
```

Snap `target_price` **down** to the store's existing price endings (read the
catalogue: .00 / .95 / .99). Amount off needs one discount per distinct
amount, so it fits small Hero sets, not large Clearance lists.

## 6. Order-level offers

Use net (ex-VAT) values throughout. `CMavg` = average full-price contribution
margin of the products that would realistically be in the basket.

### 6a. Spend-more-save-more tier (order discount `d_o` at threshold `T`)

- **Cost on a basket of exactly `T`:** `d_o · T`.
- **It pays for one customer** who would have spent `B0 < T` only if the
  extra items cover it: `CMavg · (T − B0) ≥ d_o · T`, so the customer must add
  at least `d_o · T / CMavg`. Example: CM 50%, 10% off at €150 → they must add
  ≥ €30.
- **Hidden cost, usually the biggest:** customers who'd have spent ≥ `T`
  anyway get the discount for free. Estimate the share of current orders
  ≥ `T` from recent orders. If more than ~25% of orders already clear `T`,
  the threshold is too low (SIXFIT heuristic, adjustable).
- **Placement:** start at about 1.2–1.4 × current AOV and above the median
  order (SIXFIT heuristic), then check both rules above.
- **Floor check:** a basket of exactly `T` made of the lowest-margin eligible
  products must still pass the margin floor at `d_o`.
- **Multiple tiers** (e.g. 10% at €150, 15% at €250): each tier is a separate
  discount. Keep tier spacing wide enough that the higher tier passes on its
  own.

### 6b. Free-shipping threshold

- Cost per qualifying order = your average outbound shipping cost (what the
  customer would otherwise pay, plus the subsidy you already carry).
- Same logic as 6a: the threshold must sit above most current orders, or
  it's a giveaway. Above ~25% of orders already qualifying → raise it.
- Restrict to the markets you price shipping for (`destination.countries`).

### 6c. Bundles / buy X get Y

- Effective discount = value of what's free or discounted ÷ total value of
  the items. Run the Hero test on the **bundle as a whole**.
- Best for low-return, no-size items (accessories) and natural sets.
- **Returns warning for fashion:** a partial return from a buy-X-get-Y order
  makes the refund amount ambiguous. Tell the merchant to decide and publish
  the refund rule before launch.

### 6d. Gift with purchase (good for Protect items)

- Cost = the gift's unit **cost** `c_g`, not its price. Share of the order =
  `c_g / (B / (1 + v))`.
- Protects the full price of the main item: no "% off" on bestsellers.
- Shopify doesn't add the gift to the cart automatically. The customer adds
  it and the discount makes it free. Say this, and that the product page needs
  to explain it.

## 7. VIP early access

- VIPs are mostly repeat customers who'd likely buy anyway, so a deeper VIP
  discount is mostly a subsidy. **Default: same % as public, earlier.** The
  exclusivity is the perk.
- Deeper for VIPs (+5 points) only if the merchant's priority is retention,
  and only if the product passes its test at the deeper %.
- The descending curve (guide §4) always holds: VIP ≥ public ≥ any extension.

## 8. Scenarios

Present the plan as three scenarios, then recommend one:

| | Conservative | Balanced | Aggressive |
|---|---|---|---|
| Hero % | bucket cap − 5 | bucket cap | cap + 5, only products that still pass |
| Clearance % | 30% | 40% | 50% |
| Order-level offer | none | the one that best fits the top goal | tier + free shipping |

For each scenario, show:
- products discounted per bucket
- products that fail their test (and drop out)
- average `CM(d)` on discounted products, weighted by stock
- average break-even uplift `U(d)`, and the last-year verdict if data exists
- Clearance stock value moved at cost (if it all sells)

Then recommend one, **tied to the merchant's #1 priority from the interview**
(profit → Conservative/Balanced, clearing stock → deeper Clearance, new
customers → a stronger Hero with a strict floor, AOV → a tier). Say what would
make you change the recommendation.

## 9. Honesty rules

- Every result names its weakest input ("based on your estimate of 30%
  returns").
- Inputs set to 0 because they're unknown → the sentence "these margins are
  overstated" appears next to every margin.
- Never present `U(d)` as a forecast. It's a hurdle, not a prediction.
