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
| **Item** (publicação) | The marketplace publication / **sale condition** — *how* the product is offered (price, listing type, shipping, channel). Legacy Items also carry `title`, `description`, pictures and an optional `variations[]` array. | `item_id` (`MLB…`) |
| **User Product (UP)** | New model. The seller's *specific physical product / variant*. One UP can be linked to **one or more** Items (different sale conditions). ML-assigned, not editable. | `user_product_id` |
| **Family** | ML grouping of related User Products, shown as pickers on the User Product Page. Grouped by a family calculation (PARENT_PK values identical; CHILD_PK/custom vary) — **not** just by `family_name`. `family_id` **can change** if family-defining values change. | `family_id`, `family_name` |
| **Catalog product** | Mercado Livre's canonical product page many sellers attach to and compete on (Buy Box). **Not** the same as a User Product: `catalog_product_id ≠ user_product_id`. | `catalog_product_id` (`MLB…`) |
| **Inventory location** | A physical stock allocation for a User Product (`meli_facility` / `seller_warehouse` / `selling_address`). | ML `store_id` / `network_node_id` |
| **Domain** | Cross-category product concept that drives which attributes exist and how the title/`family_name` is understood. | `domain_id` |

Full model detection, PK semantics, sale conditions, stock ownership and
multi-origin inventory: **`variations-and-user-products.md`** (the primary
architectural reference). ProductMaster and the internal `variant_id` / SKU stay
marketplace-independent; every ML id above is an **external mapping** to persist,
never Product Factory's own identity.

## Traditional model (Items + `variations[]`)  — OFFICIAL ⚠ verify

- One Item holds `title`, `price`, `description`, `pictures`.
- `variations[]` entries share the parent's price, title and description; each
  variation may have its own `attribute_combinations` (the variation axes),
  `available_quantity` (valid stock field **here**), the `SELLER_SKU` attribute,
  and `picture_ids`. (`seller_custom_field` is seller-internal only, not the
  ML SKU — see `variations-and-user-products.md` §8.)
- Max **100** variations per item (**250** Fashion / Mobile Accessories / Auto
  Parts) — a legacy-model limit, not a global cap.
- Still valid for many sellers/categories — but **do not assume it is valid for
  all**. Check account/category capability (`user_product_seller` tag).

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
- **Stock** belongs to the User Product, distributed across stock locations — not
  a single Item `available_quantity`. For multi-origin (`warehouse_management`)
  accounts `available_quantity` on `/items` is **not** the write mechanism
  (`variations-and-user-products.md` §11–§12).
- **Title** is auto-generated in the new model — the seller does not send `title`.
- Rollout: AR/MX from Oct 2024; BR + CO/CL/UY/others gradually from Jan 2025.
  A given seller may already be migrated — confirm via the `user_product_seller`
  tag, don't assume.

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

- Treating `variations[]` as universally available (BLOCKER if the seller is a
  resolved `user_product_seller`).
- Sending a manual `title` in a resolved UP flow that generates it (BLOCKER;
  unresolved model → REVIEW).
- Writing `/items` `available_quantity` for a resolved multi-origin
  (`warehouse_management`) account (BLOCKER).
- Putting variation-differentiating info only in the title/description instead of
  in `PARENT_PK` / `CHILD_PK` attributes.
- Different attribute values for the same physical trait across Item vs variation
  vs description (data conflict = BLOCKER).
- Treating `family_id` or `user_product_id` as Product Factory's own identity, or
  `catalog_product_id == user_product_id`.

## Sources

- User Products — Developers — https://developers.mercadolivre.com.br/pt_br/user-products — Developers — updated 2026-06-17 (per requester) — consulted 2026-08-27 — model objects, `family_name`/`title` mapping, UPtin, PARENT_PK grouping.
- Preço por variação — Developers — https://developers.mercadolivre.com.br/pt_br/preco-variacao — Developers — 2026 — consulted 2026-08-27 — independent offers, per-variant price/stock, 30-item limit, rollout dates.
- Publicar produtos (guia) — https://developers.mercadolivre.com.br/pt_br/publicacao-de-produtos/ — Developers — ⚠ verify — consulted 2026-08-27 — Item structure, `variations[]`.
- Multi-Origin Stock / Gestión de stock multiorigen — https://developers.mercadolibre.com.ar/en_us/multi-origin-stock — Developers (AR; MLB support DYNAMIC) — verified 2026-08-27 (search-indexed; live 403) — `warehouse_management`, `stock_locations`, UP-owned stock, `available_quantity` invalid in multi-origin. Full detail in `variations-and-user-products.md`.
- External context only (not authoritative): upseller.com, bling.com.br, ecommercenapratica.com — rollout and model summaries.
