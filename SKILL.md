---
name: shopify-black-friday
description: >-
  Black Friday / Cyber Monday pricing and discount strategy for fashion and
  apparel Shopify stores, through the connected Shopify store: which products
  to discount and which to keep at full price, how deep each discount can go
  without losing money after VAT, fees, shipping and returns, which offer type
  fits (percent off, clean price points, spend-more-save-more, free-shipping
  threshold, bundles, gift with purchase, VIP early access), EU Omnibus price
  rules, and creating the approved discounts with end dates. Use whenever a
  clothing, shoes, or accessories merchant asks what to put on sale for Black
  Friday or Cyber Monday, how much to discount, whether a sitewide sale is a
  good idea, how to price a holiday or November sale, or how to set up BFCM
  discounts. Trigger on casual phrasings too, for example "what should I put
  on sale for Black Friday", "is 30% off everything a good idea", "how deep
  should my BF discount be", "help me price my Black Friday sale", or "I'm
  late, what discounts can I still set up".
---

# Shopify Black Friday pricing

Help a fashion merchant decide **what** to discount, **how deep**, and **how**
(the offer type) for Black Friday / Cyber Monday. Base every number on their
real unit economics, then create the approved discounts in Shopify.

The biggest lever isn't the headline %. It's which products get discounted.
Blanket discounts give away margin on items that would sell anyway. Every
recommendation must show what it costs and what it needs to pay off.

## Dates

Never hardcode a year or a date. Compute them at the start of every session
from today's date:

- **Black Friday** = the day after the fourth Thursday of November of the
  current year. If this year's Cyber Monday has already passed, use next year.
- **Cyber Monday** = Black Friday + 3 days.
- **Object prefix** = `BFCM<YY>`, where `<YY>` is the last two digits of that
  Black Friday's year.

For example, from 6 Oct 2026 Black Friday is Fri 27 Nov 2026, Cyber Monday is
Mon 30 Nov 2026, and the prefix is `BFCM26`.

## What this skill does

- Interviews the merchant in detail about positioning, goals, costs, returns
  and constraints.
- Reads the store's prices, costs, sales, stock and existing discounts.
  Read-only.
- Sorts products into Protect, Hero, Clearance, Low-return boost, and Exclude,
  and computes each product's **maximum safe discount** and **break-even
  volume**.
- Recommends offer types and presents three scenarios with a clear
  recommendation.
- Creates the approved discounts, with backup, verification and rollback.

## What it does not do

- Change product prices or compare-at prices.
- Build collections, write emails or social posts, or monitor the sale.
- Forecast sales. Break-even figures are hurdles, not predictions.
- Give legal advice. EU Omnibus guidance is informational.

## How to talk to the merchant

Assume they are not technical. They run a shop.

- Plain English. No GraphQL, no GIDs, no formulas unless asked.
- Money in their currency, per sale: "you keep €21.40 per sale at 20% off,
  vs €38.10 at full price".
- Every recommendation comes with its cost and its weakest assumption.
- Show products by title, never by ID. Tables, never raw JSON.
- Lead with the date: "You have 42 days until Black Friday."

## The five phases

Work through them in order. **Finish each phase and get an explicit "yes"
before starting the next.** Silence is not approval. Load a phase's reference
file only when that phase starts.

Every run starts the same way: call `get-shop-info`, name the store back to the
merchant, compute Black Friday and the days remaining (see Dates), and confirm before
reading anything else.

---

### Phase 1 — Pricing scan

Read-only. Run the store-level pricing scan (`references/product-selection.md`
§1c): VAT-inclusive prices, markets, existing discounts and stacking, products
already on sale, cost coverage, AOV and items per order, order value
distribution, price endings, segments. Present it as a short pricing snapshot,
so the interview can confirm facts instead of asking for them.

### Phase 2 — Pricing interview

Read-only. Follow `references/pricing-interview.md`: batches A–E (positioning
and history, goals and **priority ranking**, unit economics including VAT,
fees, shipping and returns, stock and constraints, customers and offer
preferences). Ask 2–4 questions at a time, say why each matters, and skip what
the store already answers. Save `work/bfcm_pricing_brief_<YYYY-MM-DD>.json`
and read back a summary for a yes.

If the merchant opts out of discounting, Phases 3–5 become advice on non-price
offers (gift with purchase, VIP perks). Say so.

### Phase 3 — Products and safe discount depth

Read-only. Follow `references/product-selection.md` with the formulas in
`references/pricing-math.md`:

1. Pull sales (ShopifyQL, including last year's BFCM window and its
   baseline) and the full catalogue (paginated GraphQL).
2. Apply the bucket rules in order: Exclude → Protect → Clearance → Hero →
   Low-return boost → unassigned (no discount). Ask the Batch F follow-ups as
   products come up.
3. Per discounted product: contribution margin after VAT, fees, shipping and
   returns; **maximum safe discount**; **break-even uplift**; last year's
   verdict where data exists.
4. Clearance % > Hero %, or the merchant confirms why not. Clearance is
   judged on cash recovered, not full-price margin.
5. One table per bucket. The merchant moves products by name. After an
   explicit yes, save `work/bfcm_buckets_<YYYY-MM-DD>.json`.

**Cold-start:** fewer than ~30 orders in 90 days and nothing in last year's
window. Say there isn't enough history to rank products, ask the merchant
for best sellers and items to clear, and bucket on their answers plus age,
stock and size coverage (`product-selection.md` §3b).

### Phase 4 — Offer design and scenarios

Read-only. Follow `references/offer-mechanisms.md`:

1. Pick 1–3 offer types that fit the merchant's **#1 priority**, and test
   each with `pricing-math.md` (tiers and thresholds against the real order
   value distribution).
2. Present three scenarios (Conservative / Balanced / Aggressive), then
   recommend one, saying why and what would change your mind.
3. Say what you didn't recommend and why (e.g. a sitewide %, with its cost).

### Phase 5 — Create the discounts

Writes. Follow `references/discounts-and-omnibus.md`:

1. Show the stacking spec in plain English and get a yes.
2. Build the schedule (automatic discounts targeting products, order offers,
   optional VIP early access) in the shop's timezone, every discount with
   `endsAt`.
3. Validate: descending curve, one discount per product, tests re-run at
   write time on current prices, excludes, discount limit.
4. EU stores: Omnibus questions, the advertised-% rule, the quiet-period date.
   Informational, not legal advice.
5. Back up, show the plan, get a yes, apply, mark, verify. Afterwards, on
   request, confirm every discount has expired.

---

## Safety rules

Never violate these. They apply in every phase.

**Read-only until a written plan is approved.** Reading the store is fine at
any point. Writing anything requires a written plan the merchant has seen and
answered with an explicit yes. "Sounds good" to a strategy question is not
approval to write.

**Backup, plan, apply, rollback. Every write, every time.**

1. **Backup.** Immediately before writing, re-read the current state of
   everything the plan touches from the store, not from earlier in the
   conversation. Save it as `work/bfcm_backup_<run_id>_phase5.json` and offer
   it to the merchant to download. If the backup can't cover every object in
   the plan, stop.
2. **Plan.** Show what will change in plain English: a table of discounts with
   products, values, dates and audience. Then ask for a yes.
3. **Apply.** Write only what the plan lists. Check `userErrors` on every
   result, not just the first. Report progress as you go.
4. **Verify and offer rollback.** Re-read what changed, diff it against the
   plan, report what landed and what failed, and tell the merchant how to undo
   it.

**Mark everything the skill creates**, so a rollback can find it even if the
backup file is lost. Each run gets a run id, `bfcm<YY>-<YYYYMMDD>-<HHMM>` (e.g.
`bfcm26-20261006-0615`), recorded in the backup. Discounts get the title
prefix `BFCM<YY>` and a `sixfit_bfcm.created_by_run` metafield
(`single_line_text_field`, value = run id) set with `metafieldsSet`. Segments
created by the run are listed in the backup.

**Rollback finds objects by backup first, then by marker.** It deletes only
discounts and segments from this run. Anything in neither the backup nor
carrying this run's marker is never deleted or changed. Never roll back a sale
that has already taken orders: let it expire.

**Never change product prices or compare-at prices.** v1 never writes `price`
or `compareAtPrice` on any variant, by any tool. Every price reduction is an
automatic discount, written only after approval. `compareAtPrice` is read as a
signal (Clearance, Omnibus) and never written.

**Every discount has an end date.** Never create a discount without `endsAt`.
If a tool can't set an end date, don't use that tool for that discount.

**Margins are always after VAT, fees, shipping and returns.** Never show a
margin computed on a VAT-inclusive price. Any input set to 0 because it's
unknown is named next to the result: "these margins are overstated".

**The timeline is computed, never assumed.** Work from today's date against
Black Friday computed as in Dates. State the days remaining at the start of
every session. If the merchant starts late, shorten the interview (the
late-start list in `pricing-interview.md`) and say what was skipped. Never
skip a safety step.

**EU guidance is informational, not legal advice.** When Omnibus applies, say
what the rule generally requires and that the merchant should confirm with an
advisor for their country.

**Ask before guessing.** If data the plan depends on is missing (cost, VAT,
returns, handling cost, margin floor, priorities), ask. Never invent a number,
and never fill a gap with a "typical" value without saying so and getting
agreement.

**Respect exclusions for the whole run.** Products in the Exclude bucket, and
anything else the merchant names, are never added to a product discount, even
if a later step would otherwise include them.

---

## Files

| File | Purpose |
|---|---|
| `references/pricing-interview.md` | Question bank, batches, defaults. Phase 2. |
| `references/product-selection.md` | Pricing scan, data pulls, buckets, cold-start. Phases 1 and 3. |
| `references/pricing-math.md` | Contribution, max safe discount, break-even uplift, order offers, scenarios. Phases 3–5. |
| `references/offer-mechanisms.md` | Offer types, choosing by priority, interactions. Phase 4. |
| `references/discounts-and-omnibus.md` | Stacking, schedule, EU rules, create / verify / rollback. Phase 5. |
| `references/bfcm-fashion-guide.md` | Research knowledge base. All phases. |
| `work/bfcm_pricing_brief_<date>.json` | Phase 2 answers, including unknowns and the defaults used. |
| `work/bfcm_buckets_<date>.json` | Approved buckets with final discount per product. Input to Phases 4–5. |
| `work/bfcm_backup_<run_id>_phase5.json` | Pre-write state. Written before every write. |
