# Changelog

## v1.0.1 — 2026-10-20

- **VIP early access is now an optional extra.** A new interview question ("Do you have VIP
  customers?") switches it on. Shops without VIPs never see VIP questions, offers, discounts or
  segments. Shops with VIPs get a short follow-up batch: who the VIPs are, which Shopify segment,
  group size, same or deeper discount, and window length.
- The VIP window is capped at 72 hours (default 48h). Longer needs the merchant's explicit
  confirmation.
- New VIP segments are checked for members before a VIP discount is scheduled (membership takes a
  moment to compute).
- Verified on a test store: a segment-restricted automatic discount applies only to logged-in
  segment members.
- Fallback to a private discount code when the VIP list lives outside Shopify.

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
