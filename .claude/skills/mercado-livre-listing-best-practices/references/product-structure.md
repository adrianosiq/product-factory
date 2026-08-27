# Product structure — Item, User Product, Family, Catalog

last_reviewed: 2026-08-27
source_last_updated: 2026-06-17 (User Products, per requester) — ⚠ verify
volatile: true

## Why this matters

Mercado Livre is mid-transition between two publishing models. Applying the old
model's assumptions to the new one produces broken or sub-optimal listings.
Before writing content, determine **which model applies to this seller/category
right now** (DYNAMIC — confirm via API / account capabilities).

## The objects

| Object | What it is | Key ids |
|---|---|---|
| **Item** (publicação) | A single listing (`MLB…`). In the traditional model it carries title, price, description, pictures and an optional `variations[]` array. | `item_id` (`MLB…`) |
| **User Product (UP)** | New model. A seller-owned product record that centralizes several **offers / sale conditions** for the same product, each with its own price, shipping and payment terms. | `user_product_id` |
| **Family** | A group of User Products shown together as selectable *pickers* on the User Product Page. Grouped by matching `PARENT_PK` attribute values. | `family_id`, `family_name` |
| **Catalog product** | Mercado Livre's canonical product page that many sellers' listings attach to and compete on (Buy Box). Content is standardized. | `catalog_product_id` (`MLB…`) |
| **Domain** | Cross-category product concept that drives which attributes exist and how the title/`family_name` is understood. | `domain_id` |

## Traditional model (Items + `variations[]`)  — OFFICIAL ⚠ verify

- One Item holds `title`, `price`, `description`, `pictures`.
- `variations[]` entries share the parent's price, title and description; each
  variation may have its own `attribute_combinations` (the variation axes),
  `available_quantity`, `seller_custom_field` (SKU) and `picture_ids`.
- Still valid for many sellers/categories — but **do not assume it is valid for
  all**. Check account/category capability.

## New model (User Products + price per variation) — OFFICIAL ⚠ verify

- Each variation is an **independent offer** ("condição de venda" / item) with its
  own price, shipping and payment terms. Stock is per offer/variant.
- A **User Product** groups those offers for one product; a **Family** groups
  User Products that differ along `PARENT_PK` axes and displays them as pickers.
- `family_name` replaces the crafted title as the human-readable name.
  - `title` is still accepted for backward compatibility and is **internally
    mapped to `family_name`**; if both are sent, **`family_name` wins**.
  - `family_name` management is the **seller/integrator's responsibility**,
    **except UPtin**, where Mercado Livre generates `family_name` itself.
- Reported limit: **max 30 condições de venda (items) per User Product** — exceeding
  it errors. Treat the exact number as DYNAMIC / ⚠ verify.
- Rollout: AR/MX from Oct 2024; BR + CO/CL/UY/others gradually from Jan 2025.
  A given seller may already be migrated — confirm, don't assume.

## Decision flow for the agent

1. Is there a `catalog_product_id` match for this product?
   → see `catalog.md`; decide associate vs independent before writing content.
2. Is the seller/category on the **User Products** model? (DYNAMIC)
   - Yes → model as User Product(s) + Family; optimize `family_name` + attributes;
     do **not** hand-craft a title if it will be generated. See
     `variations-and-user-products.md` and `titles-and-family-name.md`.
   - No → model as Item (+ `variations[]` if multi-variant); craft the title.
3. Resolve variation axes as `PARENT_PK` (define the family / must match) vs
   `CHILD_PK` / custom (may vary within family). See `categories.md`.
4. Keep one source of truth: the `ProductMaster`. Every object's attributes,
   name, description and images must trace back to it without contradiction
   (audit dimension `CONSISTENCY`).

## Common mistakes to block

- Treating `variations[]` as universally available (BLOCKER if seller is on UP).
- Sending a marketing `title` and expecting it to survive in a UP flow.
- Putting variation-differentiating info only in the title/description instead of
  in `PARENT_PK` / `CHILD_PK` attributes.
- Different attribute values for the same physical trait across Item vs variation
  vs description (data conflict = BLOCKER).

## Sources

- User Products — Developers — https://developers.mercadolivre.com.br/pt_br/user-products — Developers — updated 2026-06-17 (per requester) — consulted 2026-08-27 — model objects, `family_name`/`title` mapping, UPtin, PARENT_PK grouping.
- Preço por variação — Developers — https://developers.mercadolivre.com.br/pt_br/preco-variacao — Developers — 2026 — consulted 2026-08-27 — independent offers, per-variant price/stock, 30-item limit, rollout dates.
- Publicar produtos (guia) — https://developers.mercadolivre.com.br/pt_br/publicacao-de-produtos/ — Developers — ⚠ verify — consulted 2026-08-27 — Item structure, `variations[]`.
- External context only (not authoritative): upseller.com, bling.com.br, ecommercenapratica.com — rollout and model summaries.
