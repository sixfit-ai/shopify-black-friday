# Black Friday / Cyber Monday Playbook for Fashion Brands on Shopify

Reference knowledge for the `shopify-black-friday` skill. Claude reads this file to make decisions; it is not shown to the merchant verbatim.

Research compiled October 2026. Re-verify dates and platform changes each season.

---

## 1. Calendar (2026)

| Event | Date |
|---|---|
| Singles' Day (pre-BF spike for global sellers) | Wed 11 Nov |
| Thanksgiving (US) | Thu 26 Nov |
| Black Friday | Fri 27 Nov |
| Small Business Saturday | Sat 28 Nov |
| Cyber Monday | Mon 30 Nov |
| Cyber Week tail | 1–3 Dec |
| Gift window (target BFCM buyers again) | 1–22 Dec |

**Treat BFCM as a ~6-week season, not a 4-day event.** Demand keeps moving earlier. Premium EU fashion brands (e.g. GANNI, Tommy Hilfiger) started their first major seasonal sales on 24 October in 2025.

Always compute the timeline relative to *today's date*. If the merchant starts late, compress it: a tight 4-week plan executed on schedule beats a 12-week plan crammed into two. Never cut the checkout test.

### Standard timeline (weeks before Black Friday)

| Window | Store / technical | Marketing |
|---|---|---|
| 8–12 weeks | Inventory forecast, app & integration inventory | Targets, last year's data |
| 6–8 weeks | Checkout audit, discount logic audit, Scripts → Functions check | Offer architecture, product selection, segments |
| 3–6 weeks | BF collection + landing page live (for indexing), page speed | Creative, email sequence, ad account warm-up |
| 2 weeks | Soft launch to 10–20% of list, support macros | List test, schedule everything |
| 1 week | Freeze: no theme changes, no new apps, no checkout edits | Final QA |
| BF weekend | Monitor | Shift budget, rotate offers |
| 30 days after | Postmortem | Post-purchase flows, gift window, reviews |

---

## 2. Set the four numbers first

Before designing any offer, the merchant must state:

1. **Revenue target** for the BFCM period.
2. **AOV target** to hold or grow.
3. **New-customer target** (acquisition is cheaper during BFCM).
4. **Margin floor**: the deepest discount they will not cross.

The margin floor must account for, together: landed cost, discount, shipping subsidy, payment fees, and **expected return rate**. Apparel holiday return rates approach ~30%; denim and dresses are the worst categories. A margin that looks fine before returns can be a loss after them.

If cost data (`inventoryItem.unitCost`) is missing in Shopify, ask the merchant for an average margin per category rather than guessing.

---

## 3. Product selection: the core decision

The biggest lever in fashion BFCM is **which products get discounted and how deeply**, not the headline percentage. Blanket sitewide discounts give margin away on items that would sell at full price anyway.

Classify every active product into one of five buckets.

### Bucket A: PROTECT (keep at full price)
- Current-season trend pieces with strong recent sell-through.
- Items that repeatedly sell out and get restocked at full price.
- New arrivals (launched in the last ~30–45 days).
- Low stock relative to recent velocity (would sell out anyway).

Rationale: industry data (Retviews/Lectra, Edited) consistently shows bestselling silhouettes holding full price through BFCM. Discounting them trains customers to wait.

### Bucket B: HERO DEALS (traffic drivers)
- Proven, broadly appealing sellers with **healthy margin** and **deep inventory across sizes**.
- These headline the campaign, go in ads and the first email.
- Moderate discount; must stay above the margin floor.
- Limit to a small set (roughly 5–15 products) so the message stays clear.

### Bucket C: CLEARANCE (deepest discounts)
- Previous-season or aged stock (e.g. no meaningful sales in 90+ days, or created long ago with high remaining inventory).
- Slow movers, including items already discounted since September.
- Broken size runs (only fringe sizes left).
- BFCM is the best moment to clear these: shoppers already expect deep cuts, which is cheaper than clearing in January and avoids cannibalising new deliveries.

### Bucket D: LOW-RETURN-RISK BOOSTS
- Accessories, bags, wallets, scarves, jewellery, gift cards: no size, lower return risk.
- In 2025, wallets and handbags showed some of the strongest year-over-year growth.
- Good for bundles and AOV threshold offers.

### Bucket E: EXCLUDE
- Products with stock below a safe threshold (risk of overselling / bad experience).
- Products with very high return rates at full price (discount amplifies the cost).
- Gift cards (from discounts), subscriptions, collaborations under contract.

### Signals Claude should use
| Signal | Source |
|---|---|
| Units sold / revenue, last 30, 60, 90 days | ShopifyQL `sales` |
| Last year's BFCM performance per product | ShopifyQL over last year's BFCM window |
| Inventory on hand per variant / size coverage | `inventoryLevels` |
| Product age | `product.createdAt` |
| Season / collection signals | product tags, product type, collections, metafields |
| Current discount state | `compareAtPrice` on variants |
| Margin | `inventoryItem.unitCost` vs. price |
| Return tendency | ShopifyQL returns / refunds by product if available |

**Velocity ratio** (sell-through proxy): units sold in last 60 days ÷ (units sold + current inventory). High = Protect or Hero candidate. Low with high stock and old createdAt = Clearance.

Claude must always show its reasoning per product in plain language and let the merchant move items between buckets before anything is written.

---

## 4. Offer architecture

**Descending curve, never ascending.** Strongest offer goes first to VIP early access, then holds or steps down through the weekend. Opening low and going deeper on Cyber Monday teaches repeat customers to wait next year.

Structures to choose from (pick 1–3, not all):
- **Bucket-based tiers**: Hero deals at X%, Clearance at Y% (Y > X), Protect at 0%.
- **Tiered by cart value**: e.g. spend more, save more. Pushes AOV without a deeper headline.
- **Bundles / outfit sets**: pair a bestseller with complementary items (strong in fashion: "complete the look").
- **Free-shipping threshold** with a visible progress bar.
- **VIP early access**: segment-restricted automatic discount, starts 24–72h before public launch.
- **Gift-with-purchase** instead of a discount for Protect items.

Rules:
- Write the intended stacking behaviour down **before** creating discounts (which discounts combine with which: product / order / shipping classes). Then test against it.
- Test the edges: two codes together, code + automatic discount, cart one cent below a threshold, gift cards, subscription items.
- Discount depth is a weaker lever than people think. Category discount peaks in 2025 ranged roughly 17–31%; going deeper than the market buys little extra attention.
- In 2025 Klaviyo reported discounts fell ~10% year-over-year while revenue still grew; repeat-buyer revenue outpaced new buyers.

### Alternative positioning
Some fashion brands opt out (Veja "Repair Friday", Freitag "Swap Friday", Armedangels, Ecoalf "Green Friday"). If the merchant positions as sustainable or premium, offer this path: no sitewide discount, VIP-only perks, donation tie-in, repair or take-back programme. The skill should ask about brand positioning before recommending discount depth.

---

## 5. EU legal: Omnibus Directive (mandatory for EU sellers)

- The reference ("was") price in any announced price reduction must be the **lowest price applied in at least the previous 30 days** (Article 6a, Price Indication Directive as amended by Directive (EU) 2019/2161).
- The advertised percentage must be computed from that 30-day low.
- Consequence: **raising prices or running another promotion in the 30 days before BF shrinks the legally showable discount.** Warn merchants not to run earlier promotions on BF products after ~28 October, unless they accept the smaller displayed discount.
- Back-to-back campaigns (BF then Christmas): the BF price may become the reference for the Christmas sale.
- Progressive, uninterrupted increasing discounts may be allowed under national derogations; national rules differ (Germany, France, Poland, etc.).
- Showing a discount against RRP or "usual price" is possible only if clearly explained.

What the skill can and cannot do:
- Shopify does not natively expose a full price history. The skill should flag every product whose `compareAtPrice` would be used as the "was" price, ask the merchant to confirm that price was not lowered in the last 30 days, and recommend an Omnibus price-history app for EU stores.
- The skill gives **informational guidance, not legal advice**. Always say so.

Also note for EU stores: the EU withdrawal-function rules (Directive (EU) 2023/2673) apply from 19 June 2026; returns/withdrawal UX should be checked during the audit.

---

## 6. Store readiness audit

Check before anything is created:
- **Shopify Scripts were sunset on 30 June 2026.** Any custom discount, shipping, or payment logic built in Script Editor stopped working silently. Compare checkout behaviour with what the merchant expects.
- Payment methods on mobile (Shop Pay, wallets, BNPL per market). Mobile carried ~57.5% of Cyber Monday 2025 sales; Shop Pay was used on ~32% of Shopify BFCM 2025 orders.
- Guest checkout enabled.
- Existing active discounts that could unintentionally stack with new ones.
- Shipping rates for each market; error states (declined card, expired code).
- Apps: remove unused ones before the freeze; confirm critical apps can handle load.
- Returns policy for sale items stated plainly on product pages; decide on extended returns window.
- Last-order dates for Christmas delivery published on-site by mid-November.
- Product schema / `priceValidUntil` correct so sale prices do not linger in search results.

---

## 7. Collections and landing pages

- Create a dedicated **Black Friday collection** and publish it early (ideally mid-October; even late it's better than on the day) so it gets indexed.
- **Keep the same URL year-round** (e.g. `/collections/black-friday`), reuse for clearance off-season. The URL accumulates authority instead of resetting each year.
- Optional sub-collections: Hero Deals, Last Chance / Clearance, Gifts Under X, VIP Early Access.
- Landing page: one lead offer above the fold, clear end date, trust signals (reviews, returns guarantee), top products only, one CTA. Never send BF traffic to the homepage.
- Urgency elements: pick at most two (e.g. countdown + free-shipping bar). Keep scarcity claims honest.

---

## 8. Segments (build before creative)

- VIPs by lifetime spend / order count
- Last year's BFCM buyers
- Recently engaged subscribers
- Lapsed customers (re-engagement)
- Browse and cart abandoners (last ~14 days)
- First-time visitors / new subscribers

Shopify customer segments can be used directly as the `context` of automatic discounts (Admin API 2025-10+), which makes native VIP early access possible.

---

## 9. Email and SMS sequence

Write and schedule everything by late October / early November. The weekend is for monitoring, not writing.

| # | Timing | Audience | Purpose |
|---|---|---|---|
| 1 | ~2–3 weeks before | All subscribers | Teaser + waitlist / early-access signup |
| 2 | ~1 week before | All | "What's coming" / gift guide angle |
| 3 | 24–72h before BF | VIP segment only | Early access live |
| 4 | BF morning | Everyone except those who already bought in early access | Public launch |
| 5 | Sat / Sun | Non-purchasers | Social proof, bestsellers, low-stock (honest) |
| 6 | Cyber Monday | Non-purchasers | Urgency, last chance |
| 7 | Tue / Wed | Non-purchasers | Close or short extension |
| 8 | Post-purchase | BFCM buyers | Order confirmation → care / styling → review request (10–14 days after delivery) → cross-sell (~3 weeks) |
| 9 | 1–22 Dec | BFCM buyers | Gift window campaign: gift sets, gift cards, price bands |

Notes:
- Exclude early-access buyers from later "now live" sends to reduce fatigue.
- Apparel brands leaned heavily into SMS in BFCM 2025; SMS works best as an amplifier of email, not a replacement.
- In fashion, include fit help (size guide link, fit notes) in every product-led email to reduce returns.

---

## 10. Instagram / social calendar

- Warm up the ad account gradually from late October; do not cold-start at 10x budget on BF morning.
- Build at least 3 creative variants per format (feed, story, reel); lock creative ~2 weeks out.
- Organic posting rhythm: teaser → countdown stories → early-access announcement → launch reel → "most loved" carousel → last-chance stories → thank-you / gift guide post.
- Use the BF collection URL as the single destination.
- Instagram and TikTok in-app browsers behave differently at checkout; test them.

---

## 11. Freeze, launch, monitor

- **One week before:** code freeze (no theme edits, app installs, checkout changes). Full purchase test on mobile + desktop, every payment method, discount combos.
- **Soft launch two weeks before:** real codes, real checkout, 10–20% of the list.
- **During the weekend watch:** revenue per hour vs. forecast, conversion rate vs. baseline, inventory burn on Hero SKUs (if selling 3x faster than forecast, shift creative to the next item), support tickets by category.

## 12. After BFCM

- All BF discounts must have an `endsAt`. Verify they expired; remove BF badges / tags / compare-at prices that should not linger.
- Decide what happens to the BF collection (keep URL, switch to clearance).
- Postmortem: which buckets, discount depths, and channels produced profit, not just revenue. Save this for next year's selection.
- Goal: second purchase from BFCM buyers within 90 days.

---

## Sources (paraphrased; check for updates)
- DigitalSuits, Shopify BFCM 2026 12-week strategy guide (Aug 2026)
- Trendtrack, Black Friday E-commerce 2026
- Lectra / Retviews fashion Black Friday reports (2024, 2025); FashionUnited; Just Style
- Edited research via FashionUnited, Retail Dive, Glossy
- Klaviyo BFCM 2025 recap and benchmarks; Klaviyo Scuffers case study
- EU Omnibus Directive summaries: K&L Gates, Sellforte, PageCrawl
- Returnalyze 2025 Peak-Season Returns report; Eightx return benchmarks; True Fit research
- FashionUnited on Green/Repair/Swap Friday alternatives
- Shopify Admin GraphQL docs: DiscountAutomaticBasicInput (2026-04 / 2026-07)
