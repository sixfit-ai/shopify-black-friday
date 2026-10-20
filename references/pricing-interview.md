# Pricing interview — Phase 2

The quality of the pricing advice depends on these answers. Ask them; never
fill them in yourself.

## How to ask

- **Batches of 2–4 questions.** Wait for the answers before the next batch.
  Never send the whole list.
- **Say why each question matters** in one short clause, so the merchant
  knows it isn't busywork ("so I don't recommend discounts that lose money
  after returns").
- **Offer answer shapes:** ranges, "roughly", "I don't know". "I don't know"
  is a valid answer. Record it, and handle it as the "If unknown" column says.
- **Read the store first** and skip what it already answers (Phase 1 scan
  data such as costs, `taxesIncluded`, markets, AOV). Confirm instead of
  asking: "Your store shows an average order of €84. Does that sound right?"
- **Follow up on surprises.** "We discounted everything 30% last year" → ask
  how full-price sales went in December and January. That tells you whether
  customers are discount-trained.
- Before moving on, read back one short summary and get a yes.

Save answers to `work/bfcm_pricing_brief_<YYYY-MM-DD>.json`, including
`"unknown"` values and the default used for each.

---

## Batch A — Positioning and history

| # | Question | Why it matters | If unknown |
|---|---|---|---|
| A1 | How would you describe your brand's price position: premium, mid-market, or value? | Premium brands lose more from deep discounts. It decides whether opting out is worth considering. | Infer from price levels, then confirm. |
| A2 | Which approach fits you: a classic discount sale, or opting out (Green Friday, repair, donation)? | Changes everything downstream. Opt-out means no % discounts at all. | Explain both in one line each, then ask again. |
| A2b | **Do you have VIP customers: a group you'd treat specially, like loyal repeat buyers, a loyalty programme, or an email list you'd give early access?** | Turns the optional VIP early-access feature on or off. Most shops don't have one, and that's fine. | **No.** VIP is skipped entirely. |
| A3 | What did you do last Black Friday (offer, depth, duration), and how did it go? | Evidence beats assumptions. It tells us the realistic uplift. | Use store data (Phase 1 scan). If none, cold-start. |
| A4 | How often do you run sales during the year, and when was the last one? | Frequent sales mean customers wait for discounts, so a deeper BF adds less. A recent sale also matters for the EU 30-day rule. | Check `compareAtPrice` and past discounts. |

## Batch B — Goals and priorities

| # | Question | Why it matters | If unknown |
|---|---|---|---|
| B1 | Revenue target for the BFCM period? | Sets the scale of the plan. | Leave blank. Advise on margin only. |
| B2 | AOV target: hold or grow it? | Decides whether a spend-more tier or free-shipping threshold belongs in the plan. | "Hold current AOV." |
| B3 | New-customer target? | Justifies a stronger Hero offer, at a strict floor. | Not a goal. |
| B4 | **Rank these by importance: profit, revenue, clearing stock, new customers, AOV.** | The recommendation follows the #1 priority. Without a ranking, any plan is a guess. | Ask again: required. |
| B5 | Your margin floor: the lowest margin you'll accept on a discounted item, after returns and costs? | Every Hero discount is tested against it. | Show their current full-price margins first, then ask again. Never invent it. |

## Batch C — Unit economics

Show any figures the store already has, and ask only for the gaps.

| # | Question | Why it matters | If unknown |
|---|---|---|---|
| C1 | Do your product costs in Shopify include freight and duties (landed cost)? If costs are missing, what's your average margin per category? | Without true costs, every margin is wrong. | Ask per category. No cost = no margin verdict. |
| C2 | Your store shows prices **including** VAT. Which VAT rates apply to your main markets? | VAT isn't your money. Ignoring it overstates margin by ~17 points at 21%. | Standard rate per market, confirmed. |
| C3 | Payment fees: roughly what % per order, plus any fixed fee per transaction? | Fees come off the discounted price too. | 0, flagged "margins overstated". |
| C4 | Shipping: what does an outbound parcel cost you, what do customers pay, and do you already have a free-shipping threshold? | The gap is a cost per order. It also tells us whether a threshold offer is realistic. | 0 subsidy, flagged. |
| C5 | Returns: what share of items come back, overall and for your worst categories (e.g. denim, dresses)? | A 30% return rate turns many "profitable" discounts into losses. | Propose 30% (guide), state it, get agreement. |
| C6 | Who pays return shipping, and what does handling a return cost you? | Each return costs money even when the item is resold. | Always ask. No default. |
| C7 | Can you resell most returns at full price, or do some come back damaged or unsellable? | Unsellable returns lose the item's cost too. | Assume all resellable, flagged. |

## Batch D — Stock and constraints

| # | Question | Why it matters | If unknown |
|---|---|---|---|
| D1 | Which items must go, how much (units or value), and by when? | Sets Clearance depth and whether liquidating below cost is acceptable. | Use store data (age, slow sellers). |
| D2 | Is new-season stock arriving soon that you need space or cash for? | Raises the value of clearing now rather than in January. | Not a factor. |
| D3 | Any products you can't discount: brand or supplier pricing rules, wholesale partners with price parity, collaborations under contract? | Discounting these can breach agreements. They go to Exclude. | None, but say this is the one to double-check. |
| D4 | Which items sell out and you restock them at full price? | They go to Protect. Discounting them gives margin away. | None named. |
| D5 | Any products already discounted since September? | They're Clearance candidates, and the EU 30-day rule affects how you can advertise them. | Check `compareAtPrice`. |

## Batch E — Customers and offer preferences

| # | Question | Why it matters | If unknown |
|---|---|---|---|
| E1 | Roughly what share of revenue comes from repeat customers? | Repeat buyers often buy anyway, which affects how deep a discount needs to go. | Use store data if there are orders. |
| E2 | Which offer types are you open to: % off, a fixed price (e.g. €99), spend-more-save-more, free-shipping threshold, bundles, gift with purchase? | Each fits a different goal (Phase 4). | Present the options in Phase 4. |
| E3 | Automatic discounts (no code), or codes for certain audiences? | Automatic converts better. Codes allow channel-specific offers. | Automatic. |
| E4 | Do you sell elsewhere (marketplaces, retail partners, your own shop) where Black Friday prices would differ? | Channel conflict and parity issues. | Online store only. |

## Batch F — Product-level follow-ups (in Phase 3)

Ask these while reviewing buckets, never as a form upfront:

- "<Product> is a Hero candidate but has only <n> units of the core sizes. Will you restock before BF, or protect it?"
- "<Product> is old and slow, but you named it as core. Clear it, or keep it at full price?"
- "<Product> needs <U>% more sales at 20% off to break even. Is its role profit or traffic?"
- "These <n> products have no cost in Shopify. What's their typical margin?"

## Batch V — VIP early access (only if A2b = yes)

Skip this whole batch when the merchant has no VIP customers. Don't suggest
VIP early access to them, and don't treat Shopify's default segments (e.g.
"Email subscribers", created in every store) as a VIP group on their own.

| # | Question | Why it matters | If unknown |
|---|---|---|---|
| V1 | Who exactly are your VIPs: repeat buyers, a spend level, a loyalty tier, your email list, or a list in another tool (Klaviyo, a loyalty app)? | Decides how Shopify can recognise them. | Ask again: required. |
| V2 | *Read the store's segments and show them.* Is one of these your VIP group, or should I create one (e.g. "customers with 2+ orders" or a customer tag)? | Shopify can only target a customer group it knows. Creating a group is a store change and needs approval. | Propose a group from V1, and show its query in plain words. |
| V3 | How many VIPs, roughly? | A very small group (under ~50) won't move much stock. The public offer has to do the work. | Count the segment members. |
| V4 | Same discount as the public sale, just earlier, or a deeper one for VIPs? | Same-but-earlier keeps margin. Deeper VIP pricing is mostly subsidy, because VIPs buy anyway (`pricing-math.md` §7). | Same %, earlier. |
| V5 | How long before Black Friday should VIP access open: 1, 2 or 3 days? | Longer windows lose exclusivity, and VIPs can buy the best stock before the public sale. | 2 days (48h). Maximum 3 days. |

**If the VIP list lives outside Shopify** (e.g. only in Klaviyo) and the
customers can't be matched to a Shopify segment, use a discount code
instead: a private VIP code with the same value and window, sent by email.
Say that codes can be shared.

## Late start

With fewer than ~10 days to Black Friday, ask only the minimum: A2, A2b, B4,
B5, C1, C2, C5, C6, D3 (plus V1, V2 and V5 if A2b = yes). Say which questions you skipped and which defaults you
used.
