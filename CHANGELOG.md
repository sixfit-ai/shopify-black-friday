# Changelog

## v1.0.0 — unreleased

First release: Black Friday / Cyber Monday **pricing** for fashion and apparel Shopify stores.

- Five phases: pricing scan, detailed pricing interview (batched, with a priority ranking),
  products and safe discount depth, offer design with three scenarios, discount creation.
- Unit economics per product: VAT stripped from VAT-inclusive prices, landed cost, payment
  fees, shipping subsidy, return rate, resellable share and return handling.
- Per product: maximum safe discount, money kept per sale, and break-even volume uplift,
  checked against last year's observed BFCM uplift when data exists. Clearance is judged on
  cash recovered, not full-price margin. Products that barely pay at full price are flagged.
- Offer types: % off tiers, amount off to clean price points, spend-more-save-more,
  free-shipping threshold, bundles / buy X get Y, gift with purchase, VIP early access by
  customer segment. Thresholds are tested against the store's real order value distribution.
- Automatic discounts with mandatory end dates, a written stacking spec, a descending-curve
  check, one discount per product, and the active automatic discount limit.
- Backup → plan → approval → apply → verify → rollback on every write. Discounts are marked
  with a `sixfit_bfcm.created_by_run` metafield.
- EU Omnibus guidance (informational): the advertised % comes from the 30-day low, the quiet
  period, the Christmas reference-price warning, and a price-history app recommendation.
- Research knowledge base: `references/bfcm-fashion-guide.md`.
