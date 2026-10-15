# Discounts and EU Omnibus — Phase 5

Writes. Every step runs behind **backup → plan → approval → apply → verify →
rollback** (SKILL.md, Safety rules).

Inputs: `work/bfcm_pricing_brief_<date>.json`, `work/bfcm_buckets_<date>.json`
(final % per product), the Phase 4 offer choice, and the Phase 1 pricing scan
(markets, existing discounts, timezone).

**v1 never writes `compareAtPrice` and never changes `price`.** Every price
reduction is an automatic discount.

---

## 1. Stacking spec (write it first, get a yes)

Before building anything, show this in plain English and ask for a yes:

> **How your Black Friday discounts combine**
> - A product gets one Black Friday discount, never two.
> - Black Friday discounts don't stack with any other product discount you
>   already run.
> - They **do** combine with shipping discounts (e.g. free shipping over €75).
> - Product deals and a spend-more-save-more offer don't stack. You'll check
>   at launch that the cart shows the price you expect.

Default `combinesWith`:

| Discount type | productDiscounts | orderDiscounts | shippingDiscounts |
|---|---|---|---|
| Product (% or amount off, BXGY, GWP) | false | false | true |
| Order (spend-more tier) | false | false | true |
| Free shipping | true | true | — |

The Phase 1 scan listed existing discounts. If an active or scheduled one overlaps the
window and has `productDiscounts: true`, name it in the spec. Shopify only
blocks stacking when **both** discounts refuse to combine, so the merchant may
need to change that discount. That's a change to an object this skill didn't
create: never do it yourself. Tell the merchant where to change it.

## 2. Structure

Automatic discounts, no codes, by default. **Target products directly**
(`items.products.productsToAdd`), grouped by bucket × final %.

| Discount | Title | Mutation | Targets / value |
|---|---|---|---|
| Hero tier | `BFCM<YY> Hero <pct>%` | `discountAutomaticBasicCreate` | Hero products at that %, ≤ 3 tiers |
| Hero clean price | `BFCM<YY> Hero €<amount> off` | `discountAutomaticBasicCreate` | `value.discountAmount { amount, appliesOnEachItem: true }`, one per amount |
| Clearance tier | `BFCM<YY> Clearance <pct>%` | `discountAutomaticBasicCreate` | Clearance products at that %, ≤ 3 tiers |
| Low-return boost | `BFCM<YY> Boost <pct>%` | `discountAutomaticBasicCreate` | Only if the merchant chose a non-zero % |
| Spend more save more | `BFCM<YY> Spend €<T> save <pct>%` | `discountAutomaticBasicCreate` | `items.all: true`, `minimumRequirement.subtotal.greaterThanOrEqualToSubtotal: T` |
| Free shipping | `BFCM<YY> Free shipping over €<T>` | `discountAutomaticFreeShippingCreate` | `minimumRequirement.subtotal`, `destination.countries` = the markets agreed |
| Bundle / GWP | `BFCM<YY> Bundle …` / `BFCM<YY> Gift …` | `discountAutomaticBxgyCreate` | buys / gets as designed in Phase 4 |
| VIP early access (optional) | `BFCM<YY> VIP <twin title>` | same as its public twin | `context.customerSegments`, % ≥ the public twin |

Every public discount uses `context: { all: ALL }`.

**Never put a product in two product discounts** (e.g. a % tier and an
amount-off). They don't combine, so the outcome depends on which one Shopify
applies. One product, one BF discount.

**Automatic discount limit.** Shopify caps active automatic discounts per
store. Shopify's dev changelog (21 Nov 2024) documents **25** for app/function
automatic discounts. The dev docs don't state a separate figure for native
basic automatic discounts, so plan against 25 total, active plus scheduled,
including the merchant's own. The Phase 1 scan counts the existing ones. The plan
(tiers × public + VIP twins + order-level offers) must fit in
`25 − existing`. If it doesn't, merge tiers, drop VIP twins for Clearance, or
drop an order-level offer, and say which. A `userErrors` limit message at
write time overrides this number: stop and report it.

### VIP early access

Only if the merchant chose a VIP-led sale, or a classic sale with early access.

- **Depth:** same % as public by default (`pricing-math.md` §7). Deeper only
  if retention is the #1 priority and the product passes its test.
- **Segment:** list existing segments and offer the obvious candidates.
  Creating a segment (`segmentCreate(name, query)`) is a write. It goes in the
  backup and gets rolled back.
- **Context:** this connector's API version supports `context.customerSegments`
  on automatic discounts (checked against the live schema). If a future
  version doesn't, fall back to a discount code (`discountCodeBasicCreate` with
  the same `context`) and tell the merchant VIPs will need the code.
- **Window:** starts 24–72h before Black Friday 00:00 (merchant picks, default
  48h). Ends **exactly** at the public `startsAt`.
- Tell the merchant: segment-restricted automatic discounts only apply to
  logged-in customers Shopify can identify. VIP emails should ask them to log
  in.

## 3. Dates

- Read `shop.ianaTimezone`. Build every `startsAt` / `endsAt` as an ISO
  timestamp with that zone's UTC offset **on that date** (e.g.
  Europe/Amsterdam is +01:00 in late November, not +02:00).
- Public window default: Black Friday 00:00:00 → Cyber Monday 23:59:59. The
  merchant can extend it to the end of Cyber Week, at the same or a lower %.
- **Every discount has `endsAt`.** Refuse to build one without it.

Example (Europe/Amsterdam, 2026): public `2026-11-27T00:00:00+01:00` →
`2026-11-30T23:59:59+01:00`. VIP (48h) `2026-11-25T00:00:00+01:00` →
`2026-11-27T00:00:00+01:00`.

## 4. Validate the whole schedule before writing

Build the full schedule as a list of `(product or "order", discount, value,
startsAt, endsAt, context)`, then check:

1. **Descending curve.** For each product, sort its discounts by `startsAt`.
   A later discount may never be deeper than an earlier one. VIP ≥ public ≥
   any extension.
2. **One BF product discount per product per moment.** VIP `endsAt` = public
   `startsAt` counts as no overlap.
3. **Tests at write time.** Re-read the current price and cost now, not from
   Phase 3, and re-run each product's bucket test (`pricing-math.md` §4) at
   the final value. Hero / Low-return must pass. Clearance must pass its floor
   or carry the merchant's recorded acceptance. Order-level offers: re-run the
   floor check on a basket of exactly `T`.
4. **Clearance % > Hero %**, or a confirmed exception recorded in
   `bfcm_buckets`.
5. **Excludes.** No Exclude-bucket or merchant-excluded product is reachable
   by any product discount. Order-level offers apply to the whole cart: tell
   the merchant that excluded products are included in the subtotal and get
   the order discount, unless they choose to drop the order offer.
6. **`endsAt` present** on every discount, and after `startsAt`.
7. **Count** within the automatic discount limit.

Any failure: stop, show it in plain English, and fix the schedule before
writing.

## 5. Omnibus (only if the Phase 1 scan found an EU market)

Informational, not legal advice. Say so every time.

- Shopify doesn't expose full price history. For each discounted product, ask
  the merchant to confirm its price wasn't lowered in the last 30 days.
  **Recommend an Omnibus price-history app to every EU store**, whether or not
  they're sure. It's the only way to show the 30-day low reliably.
- **What to advertise:** the % off must be computed from the lowest price of
  the last 30 days. Without price history, that's the current price. For
  amount-off discounts, advertise the rounded-**down** equivalent %.
- **Products that already have `compareAtPrice`:** the current price is likely
  the 30-day low, so any "X% off" must be computed from the **current** price,
  not the compare-at.
- **Quiet period:** compute Black Friday − 30 days and show it:
  > To keep your Black Friday "% off" claims clean, don't run other promotions
  > on these products from **<BF − 30 days>** onward.
  If that date has already passed, say so, and list the products that may
  have been discounted since.
- **Christmas follow-up:** if they plan a December sale on the same products,
  the Black Friday price may become the reference ("was") price for it, which
  shrinks the discount they can legally show. Plan it now.
- Close with: *This is general information about the EU price-indication
  rules, not legal advice. Check with an advisor for your country.*

## 6. Backup, apply, verify

**Backup** `work/bfcm_backup_<run_id>_phase5.json`: the existing discounts
list (ids, titles, status, `combinesWith`), the segments list, the planned
schedule, and every product's price and cost at write time. This phase changes
no existing objects, so the backup is the "before" picture plus the planned
ids.

**Apply**, per discount:

1. The create mutation from §2, with title, `startsAt`, `endsAt`,
   `combinesWith`, `context`, and `customerGets` / `minimumRequirement` /
   `destination` as designed.
2. `metafieldsSet` on the returned discount node id: `sixfit_bfcm` /
   `created_by_run` / `single_line_text_field` / run id.
3. Record the id in the backup file immediately, before the next create.
4. Check `userErrors` on both calls.

**Verify:** read back with `discountNodes(query: "title:BFCM*")`, with a
fragment per created type (`title status startsAt endsAt combinesWith context
customerGets minimumRequirement`, plus the `sixfit_bfcm.created_by_run`
metafield). Diff against the schedule: value, targets, dates, combinesWith,
context, threshold, marker. Expected status before launch: `SCHEDULED`.

**Launch check to hand the merchant:** during the window, a cart with one
Hero item and enough items to cross any tier threshold must show the price
the stacking spec promises, on mobile.

> **TODO(api 2027-01+):** add discount `tags` (e.g. `sixfit-bfcm`,
> `<run_id>`) as a second marker once the connector's Admin API version is
> 2027-01 or later. `tags` appears on discount inputs from 2027-01 (seen on
> `DiscountAutomaticAppInput`). It isn't on `DiscountAutomaticBasicInput` in
> the connector's current version. Don't implement until then. The metafield
> stays the primary marker.

## 7. After the sale

On request, or when the merchant comes back after Cyber Monday: read
`discountNodes(query: "title:BFCM*")` and confirm every discount from the run
reads `EXPIRED`. Anything still ACTIVE or SCHEDULED: name it and its `endsAt`.
Expired discounts stay as a record. Delete them only if the merchant asks.

## 8. Rollback (Phase 5)

Deletes **only** discounts and segments created by this run, found by backup
first and then by marker (`sixfit_bfcm.created_by_run` = run id, title prefix
`BFCM<YY>` as a sanity check). Restores nothing else, because Phase 5 changes
nothing else.

- Discounts: `discountAutomaticDelete(id)` (or `discountCodeDelete` for a
  code fallback).
- Segments: `segmentDelete(id)`, only if the backup lists it as created by
  this run.

Then re-read and diff against the backup. Done only when the diff is empty.
**Never roll back a sale that has already taken orders.** Let the discounts
expire instead.
