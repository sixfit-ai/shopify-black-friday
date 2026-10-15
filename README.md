# shopify-black-friday

**Black Friday pricing for fashion brands on Shopify: what to discount, how deep, and how,
worked out from your real numbers and set up in your store once you approve.**

[![Release](https://img.shields.io/github/v/release/sixfit-ai/shopify-black-friday?label=release&style=flat-square)](https://github.com/sixfit-ai/shopify-black-friday/releases/latest)
[![License: MIT](https://img.shields.io/badge/license-MIT-B0257A?style=flat-square)](LICENSE)
[![Claude Skill](https://img.shields.io/badge/Claude-skill-D97757?style=flat-square)](https://code.claude.com/docs/en/skills)
[![Shopify](https://img.shields.io/badge/Shopify-Admin%20API-95BF47?style=flat-square&logo=shopify&logoColor=white)](https://shopify.dev/docs/api/admin-graphql)

[![Website](https://img.shields.io/badge/website-sixfit.ai-B0257A?style=flat-square)](https://sixfit.ai)
[![Email](https://img.shields.io/badge/email-hello@sixfit.ai-E9D9F8?style=flat-square)](mailto:hello@sixfit.ai?subject=shopify-black-friday)

<!-- Demo: record per docs/demo-shot-list.md, convert to docs/demo.gif -->

---

## Who this is for

Brand leaders, founders and marketing managers of **fashion, shoes and accessories brands on
Shopify** who have to answer the same question every autumn: *what do we put on sale for Black
Friday, and how much?*

You don't need to be technical. You type questions in plain English, answer questions about
your business, and get a pricing plan with the reasoning behind every number. When you agree,
the discounts are created in your store for you.

---

## The problem it solves

Most Black Friday pricing is decided by gut feel: "last year we did 30%, competitors do 30%,
so 30% off everything." That approach has three hidden costs:

1. **It discounts products that would have sold anyway.** Bestsellers and new arrivals give
   away margin for no extra sales, and customers learn to wait for the sale.
2. **The real margin is much lower than it looks.** The price on the product page includes
   VAT, and then come payment fees, shipping, and returns. In fashion, holiday returns
   approach **one in three items**. A "20% off" that looks comfortable on a spreadsheet can
   lose money on every order.
3. **The headline % isn't the strongest lever.** Which products you discount, and *how* (a
   gift, free shipping over a threshold, a clean €99 price), often matters more than the
   depth.

This skill replaces gut feel with your own numbers, product by product.

---

## What you get

By the end of a session you have:

- **A pricing snapshot of your store:** your real margins after VAT, fees, shipping and
  returns, your average order, what's already on sale, and any discounts that could clash.
- **Every product sorted into a role**, with a one-line reason you can read and challenge:

  | Role | What it means | Typical treatment |
  |---|---|---|
  | **Protect** | Bestsellers, new arrivals, items you restock at full price | No discount. Maybe a gift with purchase. |
  | **Hero** | Proven sellers with healthy margin and deep stock in all sizes | The headline deal (5–15 products) |
  | **Clearance** | Old, slow, or broken-size stock | The deepest discount, judged on cash recovered |
  | **Low-return boost** | Accessories and other no-size items that rarely come back | Promote, bundle, little or no discount |
  | **Exclude** | Low stock, very high returns, supplier-protected or contract items | Never discounted |

- **For every discounted product, three numbers that matter:**
  - **Maximum safe discount:** the deepest discount that still meets your margin floor.
  - **Money you keep per sale:** at full price and on sale, after VAT, fees, shipping and returns.
  - **Break-even volume:** how many more units the discount must sell to earn the same profit
    as full price. If you have last year's Black Friday data, it says whether last year's real
    lift would have cleared that bar.
- **The right offer types for your goal**, tested against your real order values.
- **Three scenarios** (Conservative, Balanced, Aggressive) and **one clear recommendation**,
  with what would change it.
- **The discounts created in Shopify**, with end dates, only after your explicit yes, and
  reversible.

### An example of the product view

*Illustrative numbers from a test file, not a real store. 21% VAT, ~2% payment fees, 25%
margin floor.*

| Product | You keep per sale at full price | Maximum safe discount | What the discount needs to pay off |
|---|---|---|---|
| Linen Wrap Dress | €29.21 | 20% | 49% more units. Last year's Black Friday lift says it pays for itself. |
| Straight Leg Denim (38% returns) | €20.03 | 20% | 95% more units. No history, so treat with caution. |
| Wool Blend Coat (Clearance) | €36.90 | 30% (40% would lose money) | Recovers €2.84 per sale |
| Sequin Party Top ⚠ | €0.53 | none | Flagged: barely profitable even at full price, because of returns |

The last row is the kind of insight that's easy to miss without the numbers: discounting that
top would lose money on every sale, even though it looks like an obvious clearance candidate.

---

## How it works: five steps

You stay in control at every step. Each one ends with a summary you confirm before the next
begins.

### Step 1: Pricing scan *(reads your store, changes nothing)*

Claude reads your prices, costs, sales history, order values, stock and existing discounts. It
notices things like *"your prices include 21% VAT"* or *"you already have a discount that
would stack with Black Friday deals"*, so it can confirm facts with you instead of asking.

### Step 2: Pricing interview

This is what makes the advice yours rather than generic. Expect questions like these, two to
four at a time, each with a short note on why it matters:

- How would you describe your price position: premium, mid-market or value?
- Classic sale, VIP early access, or opting out of discounting altogether?
- What did you do last Black Friday, and how did it go? How often do you run sales?
- **Rank these: profit, revenue, clearing stock, new customers, bigger baskets.**
- What's the lowest margin you'll accept on a discounted item?
- Do your product costs include freight and duties? What are your payment fees? What does
  shipping cost you?
- What share of items come back, overall and in your worst categories? Who pays the return
  label?
- What must be cleared, and by when? Is new stock arriving?
- Any products you can't discount (supplier pricing rules, wholesale partners, collaborations)?
- Which items sell out and get restocked at full price?

"I don't know" is a fine answer. Claude says which default it uses instead and how that
affects the result (for example, *"I'm using 0 for payment fees, so these margins are
slightly overstated"*).

**Tip: have these ready before you start.** Rough numbers are fine:

- [ ] Product costs, or your average margin per category
- [ ] Payment fee % (your payment provider's invoice)
- [ ] What one parcel costs you to send, and what customers pay for shipping
- [ ] Return rate overall and for your worst categories
- [ ] What a return costs you (label + handling)
- [ ] Stock you need to clear, and any products you're not allowed to discount
- [ ] Your goals for the Black Friday period (revenue, average order, new customers)

### Step 3: Products and safe discount depth

Every product gets a role (see the table above) and, if it's discounted, its maximum safe
discount, money kept per sale, and break-even volume. You can move any product to another
role by name ("keep the Linen Wrap Dress at full price"), and the numbers update.

**New store or little sales history?** Claude says so plainly, asks you for your best sellers
and the items you want to clear, and builds the plan from your answers plus stock and age.

### Step 4: Offer design and scenarios

Claude chooses one to three offer types that fit your **top priority**:

| Offer type | Best when your priority is… | Example |
|---|---|---|
| % off per product | Revenue, traffic, clearing stock | "20% off our hero styles" |
| Clean price point | A simple, strong price message on a few products | €128 → **€99** |
| Spend more, save more | Bigger baskets | "10% off orders over €120" |
| Free shipping over a threshold | Bigger baskets, shipping as a barrier | "Free shipping over €75" |
| Bundles / buy X get Y | Accessories, natural sets, bigger baskets | "Scarf free with any coat" |
| Gift with purchase | Protecting full price on bestsellers | Gift on orders over €150 |
| VIP early access | Rewarding loyal customers | Same deal, 48 hours earlier |
| No discount | Premium or sustainable positioning | Repair or take-back event, VIP perks |

Thresholds are checked against your actual orders. If too many orders already qualify, the
offer is a giveaway, and Claude tells you to raise it. You then see three scenarios side by
side and a recommendation in plain words, for example: *"Balanced, because your top priority
is profit and only 4 of your 12 Hero candidates can go deeper than 20% safely."*

### Step 5: Create the discounts *(the only step that changes your store)*

1. **How discounts combine**, written in plain English, for your approval. For example: one
   Black Friday deal per product, no stacking with other product discounts, free shipping
   still applies.
2. **The full plan:** every discount, its products, value, start and end time (in your shop's
   timezone), and who gets it. Nothing is created until you say yes.
3. **Created, then checked:** Claude reads every discount back from your store and confirms it
   matches the plan.
4. **Undo is always available**, until the sale has taken orders. After that, the discounts
   simply expire on their end date.

---

## Built-in safeguards

| Safeguard | What it means for you |
|---|---|
| **Nothing changes without your yes** | Reading your store is safe. Every change needs a written plan you've seen and approved. |
| **Backup before every change** | A copy of the current state is saved first, and you can download it. |
| **One-step undo** | Everything the skill creates is marked, so it can be removed exactly, without touching anything else. |
| **Your prices are never edited** | All reductions are discounts. Product prices and compare-at prices are never changed. |
| **Every discount has an end date** | No sale accidentally running into December. |
| **No deeper deals later** | Later offers are never deeper than earlier ones, so early buyers and VIPs aren't punished, and customers don't learn to wait for Cyber Monday. |
| **No invented numbers** | Missing costs or rates are asked for, never guessed. Assumptions are named next to every result. |
| **No forecasts dressed up as facts** | Break-even volume is a bar the sale must clear, not a prediction. |

### Selling in the EU

EU rules (the Omnibus Directive) say an advertised "% off" must be measured against your
**lowest price in the previous 30 days**. Claude:

- computes the advertised % from that price, and never rounds it up
- gives you the date after which you shouldn't run other promotions on Black Friday products
- warns that your Black Friday price may become the reference price for a Christmas sale
- recommends a price-history app, because Shopify doesn't keep full price history

*This is general information, not legal advice. Check with an advisor for your country.*

---

## Setup (about 5 minutes, in claude.ai)

No API keys, no app to install, no technical setup.

1. **Turn on code execution.** In claude.ai: Settings → Capabilities → turn on *Code execution
   and file creation*. Skills don't load without it.
2. **Add the skill.**
   [Download shopify-black-friday.skill](https://github.com/sixfit-ai/shopify-black-friday/releases/latest/download/shopify-black-friday.skill),
   then go to Customize → Skills → upload the file.
3. **Connect your store.** Settings → Connectors → Shopify, and approve access for the store
   you want to price.

Custom skills are per person, so each team member who uses it uploads their own copy.

**You know it worked when** you type *"What should I put on sale for Black Friday?"* and
Claude replies with your store's name and the number of days until Black Friday.

---

## Example questions to start with

- *"What should I put on sale for Black Friday?"*
- *"Is 30% off everything a good idea?"*
- *"How deep can my Black Friday discount go without losing money?"*
- *"Should we do free shipping or spend-more-save-more this year?"*
- *"Black Friday is in 10 days, what discounts should we set up?"*
- *"We're a premium brand. Should we even discount?"*

**Moved to Shopify recently?** Share last year's Black Friday results from your old platform,
and Claude uses them as evidence for how much a discount lifts sales.

---

## Frequently asked questions

**Does it change anything in my store without asking?**
No. Steps 1–4 only read. Step 5 creates discounts only after you approve a written plan, and
every change can be undone.

**Will it change my product prices?**
Never. All reductions are Shopify discounts with an end date. Your prices and compare-at
prices are untouched.

**What if I don't know my costs or return rate?**
Say so. Claude uses a stated default where a sensible one exists (for example, ~30% returns,
typical for apparel during the holidays), asks you to confirm it, and marks every result that
depends on it. Without any cost information, a product gets no margin verdict rather than a
made-up one.

**Can I disagree with the recommendation?**
Yes, and you should whenever you know something the data doesn't. Move products between roles,
change the discount per role, or pick another scenario. Claude re-checks the numbers and tells
you what your change costs.

**Does it forecast how much we'll sell?**
No. It tells you how much extra volume each discount *needs*. If you have last year's data, it
compares that with what actually happened last year. It doesn't invent demand forecasts.

**We're a new store with few sales. Is it still useful?**
Yes. The margin math works from day one. Without sales history, Claude asks you which products
sell best and which need clearing, instead of ranking them from data.

**How late can we start?**
Ideally 6–8 weeks before Black Friday, when offer decisions are usually made. If you're late,
Claude asks a shorter set of questions and says what it skipped. The safety checks are never
skipped.

**What does it cost?**
The skill is free and MIT licensed. You need a Claude plan that supports skills and the
Shopify connector. Shopify doesn't charge for the connection.

---

## What it doesn't do

- **Collections, landing pages, emails, social posts, or ads:** it focuses on pricing.
- **Live monitoring of the sale.**
- **Sales forecasts.**
- **Legal advice:** EU guidance is informational.

### Known limitations

- **Margins are only as good as your cost data.** Missing costs mean no verdict for that product.
- **Spend-more-save-more applies to the whole cart.** Shopify can't exclude products from an
  order-level discount, so excluded items count toward the threshold. Claude points this out
  before creating one.
- **VIP early access needs customers to be logged in**, so Shopify can recognise them. Tell VIPs
  to log in.
- **Product deals combined with a spend-more offer:** check one test cart on mobile when the
  sale starts, to confirm the price matches the plan.
- **Shopify limits active automatic discounts to 25 per store**, including your own. The plan
  is kept within that limit.

---

<details>
<summary><b>For maintainers</b></summary>

<br>

**Repository layout**

| Path | Contents |
|---|---|
| `SKILL.md` | The workflow and safety rules Claude follows |
| `references/pricing-interview.md` | The question bank and defaults |
| `references/product-selection.md` | Data pulls, pricing scan, bucket rules, cold-start |
| `references/pricing-math.md` | Margin, maximum safe discount, break-even volume, order offers, scenarios |
| `references/offer-mechanisms.md` | Offer types and how to choose them |
| `references/discounts-and-omnibus.md` | Stacking, schedule checks, EU rules, create / verify / rollback |
| `references/bfcm-fashion-guide.md` | Research knowledge base |

**Building the bundle.** The `.skill` file is a zip whose top-level folder is the skill name.
It's committed to the repo and attached to each GitHub release.

```bash
cd .. && rm -f shopify-black-friday/shopify-black-friday.skill && \
  zip -r -X shopify-black-friday/shopify-black-friday.skill \
    shopify-black-friday/SKILL.md shopify-black-friday/references -x '*.DS_Store'
```

**Workflows.** `download-tracker.yml` needs the `SLACK_WEBHOOK_URL` secret and the
`download-stats` branch; `star-fork-notify.yml` needs `SLACK_WEBHOOK_URL`.

</details>

## Support

Questions, or a recommendation that doesn't look right? Open an issue at
[github.com/sixfit-ai/shopify-black-friday/issues](https://github.com/sixfit-ai/shopify-black-friday/issues)
or email [hello@sixfit.ai](mailto:hello@sixfit.ai?subject=shopify-black-friday). If you're
reporting a pricing result, include the product, its price and cost, and the answers you gave
about VAT, fees and returns. That's usually enough to see what happened.

## License

MIT — see [LICENSE](LICENSE). Built by [SIXFIT.AI](https://sixfit.ai).
