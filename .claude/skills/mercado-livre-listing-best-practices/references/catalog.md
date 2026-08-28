# Catalog

last_reviewed: 2026-08-27
volatile: true

## What the catalog is (OFFICIAL ⚠ verify)

- Mercado Livre's **catalog** groups every listing of an *identical* product onto
  one canonical product page identified by a **`catalog_product_id`** (`MLB…`).
- When your listing is **linked** to that catalog product, it stops being a
  standalone page in search and instead **competes on the catalog page** with
  every other seller linked to the same product.
- The winning offer gets the **"vendedor em destaque" / Buy Box** position.

## Catalog vs seller-controlled content

| Controlled by the catalog (standardized) | Controlled by the seller |
|---|---|
| Product name / title | Price |
| Ficha técnica / core attributes | Stock & availability lead time |
| Main images | Shipping (mode, cost, Full) |
| Description (canonical) | Listing type (Clássico / Premium) |
| | Reputation, operational quality, policies |

So: for a catalog listing, do **not** spend effort crafting an independent title
or description — those come from the catalog. Focus the effort on the commercial
levers and on making sure you're linked to the **correct** catalog product.

## Decision flow (run at workflow step 4, before writing content)

1. Search the catalog for the product (GTIN, brand+model, key attributes):
   `GET /products/search` / domain catalog search → candidate `catalog_product_id`.
2. **Exact match found?**
   - Yes → recommend **associating to catalog**. Verify the match is truly the
     same product (same model, capacity, color scope, edition). A near-match is a
     wrong match — flag it.
   - No → create an **independent listing**; still fill attributes and the
     product identifier (where one legitimately exists — `attributes.md`) well so
     a future catalog product can match it.
3. Some categories/products **must** go through catalog (catalog-mandatory
   domains) — check the category settings (DYNAMIC). If mandatory and no match
   exists, escalate.
4. If associating: fill `catalog_required` attributes; expect title/description/
   main images to be catalog-driven.

## Buy Box levers (mix of OFFICIAL ⚠ verify + EXPERIMENTAL)

Commonly cited: competitive price, seller reputation, shipping (free / Full /
speed), listing type, stock availability, dispatch lead time, and operational
quality. Exact weighting is not published — treat prioritization as EXPERIMENTAL
and confirm against current ML docs.

## Audit checks (`CATALOG`)

- [ ] Catalog searched before independent content was written.
- [ ] If linked: `catalog_product_id` is an exact product match (model, capacity,
      color scope, edition), not a near-match.
- [ ] If linked: no wasted effort on independent title/description; commercial
      levers addressed instead.
- [ ] If catalog-mandatory domain: listing is linked or the gap is escalated.
- [ ] `catalog_required` attributes filled when linked.

## Sources

- Identificadores de produtos — https://developers.mercadolivre.com.br/pt_br/identificadores-de-produtos — Developers — ⚠ verify — consulted 2026-08-27 — `catalog_product_id`, catalog matching.
- Publicar produtos (guia) — https://developers.mercadolivre.com.br/pt_br/publicacao-de-produtos/ — Developers — ⚠ verify — consulted 2026-08-27 — catalog listings.
- External context only (not authoritative): ideris.com.br, upseller.com, vencebox.com.br — Buy Box levers and catalog behavior (EXPERIMENTAL/INTERNAL).
