# Offer mechanisms — Phase 4

Choose **how** to discount, not just how much. Pick 1–3 mechanisms (guide
§4), never all of them. Every mechanism is checked with
`pricing-math.md` before it's recommended.

## 1. Menu

| Mechanism | Best for | Avoid when | Created with |
|---|---|---|---|
| **% off per product** (bucket tiers) | Hero and Clearance; a clear, simple message | Products near the margin floor | `discountAutomaticBasicCreate`, `customerGets.value.percentage`, `items.products` |
| **Amount off per item → clean price** (€128 → €99) | Small Hero sets where the price point sells | Large lists (one discount per amount) | `discountAutomaticBasicCreate`, `value.discountAmount { amount, appliesOnEachItem: true }` |
| **Spend-more-save-more** (order % at threshold) | The AOV goal; stores with many low-price items | Many orders already above the threshold; low margins | `discountAutomaticBasicCreate`, `items.all: true`, `minimumRequirement.subtotal` |
| **Free-shipping threshold** | The AOV goal; markets where shipping is a known barrier | Shipping is already free; heavy or remote parcels | `discountAutomaticFreeShippingCreate`, `minimumRequirement.subtotal`, `destination.countries` |
| **Bundle / buy X get Y** | Low-return accessories, natural sets, AOV | Sized garments with high returns (partial-return refunds get messy) | `discountAutomaticBxgyCreate` |
| **Gift with purchase** | Protect items: rewards buying without cutting the price | The gift's cost is high, or stock is thin | `discountAutomaticBxgyCreate` (gift 100% off), or basic with a minimum |
| **VIP early access** | VIP-led positioning, retention | No identifiable VIP group; customers rarely log in | Any of the above with `context.customerSegments` |
| **No discount (opt-out)** | Premium / sustainable positioning (guide §4) | — | Nothing. Only the advice. |

## 2. Choosing, by #1 priority (interview B4)

| #1 priority | Lead mechanism | Add only if it passes | Never |
|---|---|---|---|
| **Profit** | Few Hero products at `d*`, Clearance at the clearance floor | GWP on Protect items | Sitewide %; a tier with > 25% of orders already above it |
| **Clearing stock** | Deep Clearance %, by tier | A bundle pairing a slow item with a bestseller (bestseller at full price) | Discounting Protect items to "pull traffic" to Clearance |
| **New customers** | Strong Hero % on proven, deep-stock items | Free-shipping threshold set near current AOV | VIP-only offers (existing customers only) |
| **AOV** | Spend-more-save-more or a free-shipping threshold | Bundles in low-return categories | Deep per-product % (lowers basket value) |
| **Revenue** | Balanced Hero + Clearance | A tier, if it passes 6a | Going deeper than the market's 17–31% peak band (guide §4): little extra attention |

**Store-type adjustments:**
- **Premium / discount-sensitive brand:** propose opt-out or VIP-only first.
  If they still want a sale, keep Hero ≤ 20% and prefer GWP.
- **Customers discount-trained** (frequent sales, A4): a deeper BF adds less.
  Prefer mechanisms that aren't a plain % (tier, GWP, bundle).
- **High return rate (> 30%) in a category:** prefer amount off on fewer
  products, or bundles of no-size items. Never deep % on that category.
- **Cold-start store:** no evidence for uplift. Prefer Conservative, and say
  why.

## 3. Interactions (design before creating)

- **One BF discount per product.** Products in a % discount aren't also in an
  amount-off discount.
- **Product vs order discounts:** by default BF product discounts don't
  combine with order discounts (stacking spec). The dev docs don't state which
  discount Shopify applies when several eligible ones can't combine, so
  **don't rely on it**. Add a launch check: a cart with a Hero item that also
  crosses the tier threshold must get the price you expect. Test it with a
  checkout preview.
- **Free shipping combines with everything** by default
  (`shippingDiscounts: true` on product and order discounts).
- **Cap:** stay within the active automatic discount limit (25; see
  `discounts-and-omnibus.md` §2). Count before proposing.

## 4. Presenting

For each recommended mechanism, give: what the customer sees ("€99 instead
of €128"), what it costs you, the result of its test, and **one sentence on
why it fits their #1 priority**. Then the three-scenario table
(`pricing-math.md` §8), and a clear recommendation.

Also say what you **didn't** recommend and why, in one line each. Merchants
often ask for a sitewide discount. Explain the cost instead of just refusing.
