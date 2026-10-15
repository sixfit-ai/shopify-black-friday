# Product selection — Phase 3

Sort every active product into one of five buckets, or leave it unassigned.
Read-only. Nothing in this phase writes to the store.

Source of truth for the strategy behind the buckets:
`references/bfcm-fashion-guide.md` sections 2–6.

Thresholds below come from the guide where it gives a number. Values marked
"(SIXFIT default, adjustable)" are ours: the guide gives no figure for them.

---

## 1. Data to pull

Pull everything before scoring anything. Save raw results under `work/`.

### 1a. ShopifyQL (`run-analytics-query`)

Net units and net sales by product, three windows:

```
FROM sales SHOW net_items_sold, net_sales, orders
GROUP BY product_title SINCE -30d UNTIL today ORDER BY net_sales DESC
```

Repeat with `SINCE -60d` and `SINCE -90d`.

Last year's BFCM window: the week before last year's Black Friday through last
year's Cyber Monday. Compute last year's Black Friday with the same rule as in
SKILL.md (e.g. 28 Nov 2025 → window `2025-11-21` to `2025-12-01`):

```
FROM sales SHOW net_items_sold, net_sales, orders
GROUP BY product_title SINCE 2025-11-21 UNTIL 2025-12-01 ORDER BY net_sales DESC
```

Baseline for observed uplift `L` (`pricing-math.md` §3): the 4 weeks before
last year's window, e.g. `SINCE 2025-10-24 UNTIL 2025-11-20`, same columns,
grouped by product. Divide by 4 × (window days ÷ 7) to get a same-length
baseline.

Returns by product, last 90 days:

```
FROM sales SHOW quantity_returned, returns, net_items_sold
GROUP BY product_title SINCE -90d UNTIL today
```

Return rate per product = `quantity_returned / (net_items_sold + quantity_returned)`.

**Last year's results from elsewhere.** If the store is new on Shopify or
history is thin, ask whether the merchant has last year's BFCM results
(an export from their old platform, or notes on units per product). Use them
as the last-year signal for `L`, and say you're doing so.

### 1b. Admin GraphQL (`graphql_query`, paginated)

Products:

```graphql
query Products($after: String) {
  products(first: 50, after: $after, query: "status:active") {
    pageInfo { hasNextPage endCursor }
    nodes {
      id title status createdAt productType tags
      variantsCount { count }
      options { name values }
    }
  }
}
```

Variants, through the top-level connection, not `product.variants`:

```graphql
query Variants($after: String) {
  productVariants(first: 250, after: $after) {
    pageInfo { hasNextPage endCursor }
    nodes {
      product { id }
      price compareAtPrice inventoryQuantity
      selectedOptions { name value }
      inventoryItem { tracked unitCost { amount } }
    }
  }
}
```

Follow `endCursor` until `hasNextPage` is false. Then check that the variant
total equals the sum of `variantsCount`. If it doesn't, stop.

### 1c. Store-level pricing scan (Phase 1)

Read once, before scoring. Report it to the merchant as a short pricing
snapshot, not as a pass/fail audit.

| Item | How | Used for |
|---|---|---|
| Prices include VAT? | `shop { taxesIncluded currencyCode ianaTimezone }` | VAT strip (`pricing-math.md` §1). Confirm rates in the interview. |
| Markets / EU | `markets { nodes { name status conditions { regionsCondition { regions { nodes { ... on MarketRegionCountry { code } } } } } } }` | Omnibus on or off, VAT rates to ask, free-shipping destinations |
| Existing discounts | `discountNodes` with `title status startsAt endsAt combinesWith` per type | Stacking risk; automatic discounts in use (limit 25) |
| Products already on sale | Variants with `compareAtPrice` set | Clearance signal; Omnibus advertising base |
| Cost coverage | Share of active variants with `inventoryItem.unitCost` | Which products can get a margin verdict |
| AOV and items per order | `FROM sales SHOW orders, net_sales, net_items_sold, average_order_value SINCE -90d UNTIL today` | `n`, tier and threshold placement |
| Order value distribution | `orders(first: 250, query: "created_at:>=<today−90d>") { nodes { subtotalPriceSet { shopMoney { amount } } } }`, paginated | Median order; share of orders above a candidate threshold (`pricing-math.md` §6a–b) |
| Price endings | The last two digits of variant prices (.00 / .95 / .99) | Clean price points (`pricing-math.md` §5) |
| Customer segments | `segments(first: 50) { nodes { id name query } }` | VIP early access candidates |

### 1d. Known quirks

- **Blank MONEY cells.** ShopifyQL can return `""` instead of `0` for MONEY
  fields on products with zero orders. Coerce to 0 before any math. Never show
  a blank as a number to the merchant without saying it means "no sales".
- **Empty grouped results.** A `GROUP BY` query over a window with no sales
  returns `columns: []` and `rows: []`, not an error. Invalid column names do
  error, so an empty result means no sales in the window. Confirm with the same
  query without `GROUP BY`, which returns a single row of zeros.
- **`search_products` silently ignores filters** and returns the whole catalog.
  Never use it for anything that drives a decision. Use the paginated GraphQL
  queries above.
- **`product.variants(first: N)` truncates silently.** Bundles with option
  combinations easily exceed 50 or 100 variants. Always use
  `productVariants` with pagination, and reconcile against `variantsCount`.
- **Response size.** Large variant pulls overflow the tool response. Request
  only the fields above, use short aliases if needed, and process results in a
  file rather than reading them inline.

---

## 2. Derived metrics

Compute per product. Show the merchant numbers in plain words, not formulas.

| Metric | Formula | Notes |
|---|---|---|
| **Velocity ratio** | `units_60d / (units_60d + current_inventory)` | 0 = nothing sold, 1 = sold through. Undefined if inventory is untracked on all variants. |
| **Age** | `today − createdAt`, in days | |
| **Size coverage** | share of the product's size values with `inventoryQuantity > 0` on at least one variant | Size option = any option whose name normalizes to size (`Size`, `Dress Size`, `X (Size)`). No size option → 1.0. |
| **Contribution `C(d)`, margin `CM(d)`, break-even uplift `U(d)`, max safe discount `d*`** | See `pricing-math.md` §2–4 | VAT stripped. Fees, shipping subsidy, returns and handling included. Inputs come from the Phase 2 brief. |
| **Observed BFCM uplift `L`** | `pricing-math.md` §3 | Needs last year's window plus the 4 weeks before it (query in §1a). |

**Cost** comes from `inventoryItem.unitCost`, or the merchant's average margin
per category from the brief. If neither exists, the product can't enter Hero
or Clearance. Never invent a cost. All other inputs (VAT, fees, shipping,
returns, handling, resellable share) and their "if unknown" handling are in
`pricing-math.md` §1.

---

## 3. Bucket rules

Evaluate in this order. **The first rule that matches wins.** A product never
lands in two buckets.

1. **Exclude**
2. **Protect**
3. **Clearance**
4. **Hero**
5. **Low-return boost**
6. **Unassigned** — no discount. This is the default.

### Default thresholds

All values are adjustable per store. They are defaults, not rules of nature.
Small catalogs (under ~50 active products) or young stores (under ~6 months of
sales) should relax the unit minimums. Say so when you adjust.

| Bucket | Matches when | Default |
|---|---|---|
| **Exclude** | Merchant named it. Gift card, sample, test, or non-physical item. Not ACTIVE. No sellable stock (tracked inventory 0 on all variants). | — |
| | **Proposed, merchant confirms:** low stock, total tracked inventory < **L** units (oversell / bad-experience risk). | L = 5 (SIXFIT default, adjustable) |
| | **Proposed, merchant confirms:** very high full-price return rate, ≥ **X** × store average with ≥ 10 units sold in 90 days (a discount amplifies the cost). | X = 1.5 (SIXFIT default, adjustable) |
| **Protect** | New arrival: age < **N** days. | N = 45 (guide: ~30–45 days) |
| | Would sell out at full price: velocity ratio ≥ **V**. | V = 0.6 (SIXFIT default, adjustable) |
| | Merchant-named core / never-discount line. | — |
| | Merchant-named restock items. Ask in Phase 3: *"Which items sell out and you restock them at full price?"* Restocks aren't detected from the API. | — |
| **Clearance** | Dead stock (guide rule): no meaningful sales in the last 90 days AND age ≥ 90 days AND stock on hand. "No meaningful sales" = ≤ **M** net units in 90 days. | M = 2 (SIXFIT default, adjustable) |
| | Old and slow: age ≥ **A** days AND velocity ratio ≤ **S**. | A = 180, S = 0.15 (SIXFIT default, adjustable) |
| | Broken sizes: size coverage ≤ **C**. | C = 0.4 (SIXFIT default, adjustable) |
| | Already on sale (`compareAtPrice` set on any variant). Treat as a strong signal, and confirm with the merchant. | — |
| **Hero** | Proven demand: top by net units in last year's BFCM window, else last 60 days. Size coverage ≥ **H**. Inventory ≥ **I** units. Passes the Hero test at the proposed discount (`pricing-math.md` §4). | H = 0.7, I = 20 (SIXFIT default, adjustable) |
| | **Cap: 5–15 products.** Default cap = `clamp(round(active_products × 0.05), 5, 15)`. Take the top candidates by units; the rest fall through. | |
| **Low-return boost** | **Either** the category rule (guide): no size option, OR product type / tags match accessories, bags, wallets, scarves, jewellery (case-insensitive, EN + the store's language). Works in cold-start. | — |
| | **Or** the return-rate rule: return rate ≤ **R** × store average AND ≥ **U** units sold in 90 days. | R = 0.5, U = 10 (SIXFIT default, adjustable) |
| | Either way: promote it (placement, email, ads, bundles, AOV thresholds), and give it little or no discount. | |
| **Unassigned** | Everything else. Sells at full price. | — |

"Stock on hand" = tracked inventory > 0 on at least one variant.

**Why Clearance dead stock uses AND (age ≥ 90 days) where the guide says
"or":** AND keeps new products with no sales history out of Clearance. The
guide's other case, "old with high inventory", is covered by the old-and-slow
rule.

**Clearance is judged on cash, not full-price margin.** Default clearance
floor: `C(d) ≥ 0` (every sale at least pays its own costs). Dead stock's real
alternative is a deeper January markdown or a write-off, so comparing it to
full-price profit is the wrong test. Products that fail at the bucket % are
flagged with their `d*`. The merchant either lowers to `d*` or explicitly
accepts selling below cost (a negative clearance floor), product by product
or for the whole bucket.

**Hero and the margin floor.** No exceptions. A Hero product that can't pass
the Hero test at ≥ 10% isn't a Hero. It drops through to the next rule.

**Missing data never promotes a product.** No cost means no Hero and no
Clearance. No sales history means no Hero and no return-rate Low-return boost
(the category rule still applies). Untracked
inventory means no Hero (stock depth can't be checked) and no Clearance on the
old-and-slow or broken-sizes rules (velocity and size coverage are undefined).

**Exclude by name is a proposal.** Titles that look like test, sample, or
placeholder items ("TEST", "SAMPLE", "DUMMY", "DO NOT USE") are proposed for Exclude
and confirmed by the merchant. Never exclude silently on a title match.

### Proposed discount per bucket

| Bucket | Default discount | Notes |
|---|---|---|
| **Hero** | 20% (cap; per product ≤ `d*`) | The guide notes 2025 category discount peaks ran ~17–31%. Deeper buys little extra attention. |
| **Clearance** | 40% | Deep on purpose: the goal is to move the stock. Tested against the clearance floor. |
| **Low-return boost** | 0% (range 0–10%) | Promote instead of discounting. If discounted, same test as Hero. |
| Protect, Unassigned, Exclude | none | |

The merchant can change the percentage per bucket. **Whenever they do,
check Clearance % > Hero %.** If it isn't, explain: *"Your clearance items
would be discounted less than your headline deals. Shoppers then see the
best offers on your best products, and your old stock doesn't move."* Ask
them to confirm or adjust. Record a confirmed exception in `bfcm_buckets`.

Then, per product:

- **Hero:** if the product fails the Hero test at the bucket %, lower its
  discount in **5-point steps** (20 → 15 → 10) to its `d*`. If it can't clear at ≥ 10% (SIXFIT default, adjustable), it isn't a
  Hero. It drops to the next rule. Show every product that was lowered.
- **Tiers:** at most **3 distinct percentages per bucket**, all multiples of
  5. If lowering would create a 4th tier, round the extra products down into
  the nearest lower tier that still clears the floor, or drop them from the
  bucket.
- **Clearance:** never lower automatically. Flag products that fail the
  clearance floor, show `d*`, and let the merchant choose (see above).
- **Every discounted product** also gets its break-even uplift `U(d)` and,
  if data exists, the last-year verdict. Show both in the Key numbers column.
- **Low-return boost:** same as Hero if the merchant chose a non-zero %.

Save the final percentage per product as `discount` in `bfcm_buckets`.
Phase 4 chooses the offer mechanisms (`offer-mechanisms.md`). Phase 5 groups
products by bucket and percentage into discounts
(`discounts-and-omnibus.md`).

---

## 3b. Cold-start mode

**Trigger:** fewer than ~30 orders in the last 90 days AND no sales in last
year's BFCM window. Check with the ungrouped queries:

```
FROM sales SHOW orders SINCE -90d UNTIL today
FROM sales SHOW orders SINCE <last_bf − 7d> UNTIL <last_cm>
```

In cold-start:

1. **Say it plainly:**
   > Your store doesn't have enough sales history yet for me to rank products
   > automatically. I'll ask you a few questions and use what the store can
   > still tell me: how old each product is, how much stock you have, and
   > which sizes are still available.
2. **Ask, a few at a time, by title:**
   - "Which 3–5 products sell best, or get the most interest?"
   - "Which 3–5 products do you most want to clear?"
   Match titles case-insensitively. If more than one product matches, ask which one.
3. **Use what the store still knows:** age, stock depth, size coverage,
   `compareAtPrice` already set.
4. **Apply the same evaluation order**, with these substitutions:
   - **Hero candidate** = merchant-named best seller AND size coverage ≥ H
     AND inventory ≥ I. The Hero test still applies. The cap still applies.
   - **Clearance candidate** = merchant-named slow item, OR age ≥ 90 days
     AND inventory in the top quarter of the catalog, OR `compareAtPrice`
     already set. The clearance floor still applies (flag with `d*`, don't lower).
   - **Protect** = new arrival (age < 45 days), or merchant-named core.
   - **Low-return boost**: category rule only (no size option, or accessory
     type / tags). The return-rate rule is skipped: there's no return data.
   - **Protect** also includes the items the merchant names as full-price
     restocks.
   - Everything else is Unassigned.
5. **In the Why column, start every cold-start decision with "Based on what
   you told me:"**, or "Based on stock and age:" when it came from store data
   only.

Record `"mode": "cold_start"` and the merchant's named lists in
`bfcm_buckets`.

---

## 4. Showing the result to the merchant

Start with a short summary:

> Out of 240 active products: **8 Hero**, **31 Clearance** (€14,200 of stock
> at cost), **22 Protect**, **12 Low-return boost**, **5 Exclude**. The other
> 162 stay at full price.

Then show one table per bucket, in this order: Hero, Clearance, Low-return
boost, Protect, Exclude.

| Product | Type | Key numbers | Why |
|---|---|---|---|
| Linen Wrap Dress | Dresses | 84 sold last BFCM · 140 in stock · 52% margin at 25% off | Last year's top BFCM seller, deep stock, still profitable on sale. |

- **Key numbers:** the two or three that drove the decision, in words.
- **Why:** one line, plain English. No metric names.
- **Large catalogs** (more than ~20 products in a bucket): show the top 10 per
  bucket and offer the full list as a downloadable CSV,
  `work/bfcm_buckets_<date>.csv`.

## 5. Merchant changes

The merchant moves products by name: "move the Linen Wrap Dress to Protect."

- Match by title, case-insensitively. If more than one product matches, ask
  which one.
- Moving **into Hero or Clearance**: re-run that bucket's test and show `d*`
  and `U(d)` before accepting. Moving into Hero also re-checks the cap.
- Moving into Exclude always succeeds and holds for the whole run.
- After changes, show the updated summary and ask for explicit approval.

## 6. Saving

After explicit approval, write `work/bfcm_buckets_<date>.json`:

```json
{
  "shop": "example.myshopify.com",
  "approved_at": "2026-10-06T10:15:00+02:00",
  "black_friday": "2026-11-27",
  "mode": "normal",
  "bucket_discounts": { "hero": 0.20, "clearance": 0.40, "low_return": 0.0 },
  "merchant_named": { "best_sellers": [], "to_clear": [] },
  "thresholds": { "protect_new_days": 45, "...": "..." },
  "buckets": {
    "hero":        [{ "id": "gid://shopify/Product/1", "title": "...", "reason": "...", "discount": 0.20, "lowered_from": null }],
    "clearance":   [],
    "low_return":  [],
    "protect":     [],
    "exclude":     [],
    "unassigned":  []
  },
  "merchant_overrides": [{ "title": "...", "from": "hero", "to": "protect" }]
}
```

Phases 4 and 5 read this file. Never regenerate it silently. If the catalog
changes before Phase 5, re-run Phase 3 and ask for approval again.
