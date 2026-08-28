# Pricing & commercial conditions

last_reviewed: 2026-08-27
volatile: true

This Skill does not set the final price — that's a business decision. It ensures
the commercial setup is coherent, competitive-aware and correctly classified.

## Missing commercial inputs are not a content blocker

- Acquisition/landed cost, target margin and target price are
  **COMMERCIAL_OPTIONAL** (`SKILL.md` §2 D). If they are absent, still produce the
  full listing content (title/family, attributes, description, image strategy) and
  emit `WARNING — pricing/profitability analysis unavailable`. Never FAIL or
  block the draft for a missing cost or margin.
- Price, currency and listing type are **PUBLICATION_REQUIRED** (`SKILL.md` §2 C):
  a gap there is a publication-readiness gap (`REVIEW`), resolved before the
  separate publish step, not during content drafting.
- Stock / availability and handling time are PUBLICATION_REQUIRED for the offer,
  but real per-variant stock is also needed to decide whether to model a variant
  at all (see below and `variations-and-user-products.md`).

## Classify every commercial statement

| Class | Example |
|---|---|
| **Requirement (OFFICIAL)** | New-condition items in some sites require immediate payment; catalog-mandatory domains must be linked. |
| **Official recommendation (OFFICIAL)** | ML recommends competitive pricing, good photos, clear title, offering installments/free shipping where viable. |
| **Commercial strategy (INTERNAL / EXPERIMENTAL)** | "Start Clássico, move winners to Premium"; "match the 3rd-lowest catalog price". |

Never phrase a strategy as an ML rule.

## Listing types (OFFICIAL ⚠ verify)

- **Grátis** — limited exposure/quantity, no commission (limited eligibility).
- **Clássico** — standard exposure; commission applies.
- **Premium** — highest exposure; commission applies; supports installments
  without interest to the buyer.
- Commission ranges roughly **10–19%** by category, plus a possible fixed fee on
  low-priced items. Confirm the **current** rate for the category (DYNAMIC).
- Choose per product based on margin and expected volume — this choice is
  INTERNAL strategy, not an ML rule.

## Price per variation (new model)

- In the User Products model, price/shipping/payment can differ per sale
  condition/variant (see `variations-and-user-products.md`).
- Keep per-variant prices explainable (e.g. larger size costs more) — unexplained
  spreads confuse buyers and invite questions/returns.

## Shipping (OFFICIAL ⚠ verify + INTERNAL)

- Free-shipping cost to the seller scales with **reputation** and price band
  (better reputation → larger ML subsidy).
- Offering free shipping and fast/Full shipping is an OFFICIAL recommendation for
  competitiveness; whether it's worth it for a given SKU is INTERNAL math.
- Set an honest **availability lead time** / handling time. Overstating dispatch
  speed is a top driver of complaints — see `return-prevention.md`.

## Stock

- Real stock only. Per-variant in both models.
- Don't publish variants that are permanently out of stock — they hurt the
  listing's quality signals and buyer trust.

## Do not

- Claim a pricing/shipping/listing-type choice "boosts ranking" as if ML said so.
- Put price, installment or shipping promises in the **title** (prohibited) or
  description.
- Promise interest-free installments that the listing type doesn't provide.

## Audit checks (part of `COMPLIANCE` + `CONSISTENCY`)

- [ ] Listing type consistent with the installment/shipping claims made.
- [ ] New-condition payment requirements satisfied for the site.
- [ ] Per-variant price differences are explainable.
- [ ] Lead time / handling time is realistic.
- [ ] No commercial claims stated as ML ranking rules.
- [ ] No price/promo/shipping text in the title.

## Sources

- Como criar anúncios eficientes — https://vendedores.mercadolivre.com.br/nota/como-criar-anuncios-eficientes-no-mercado-livre — Central de Vendedores — 2026 ⚠ verify — consulted 2026-08-27 — competitive price / installments / shipping as levers.
- Publicar produtos (guia) — https://developers.mercadolivre.com.br/pt_br/publicacao-de-produtos/ — Developers — ⚠ verify — consulted 2026-08-27 — immediate payment for new items (site-specific).
- External context only (not authoritative): olist.com, upseller.com, ecommercenapratica.com — listing-type exposure tiers and commission ranges (confirm current values via ML).
