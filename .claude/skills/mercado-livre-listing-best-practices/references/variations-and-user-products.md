# Variations, User Products & multi-origin inventory

last_reviewed: 2026-08-27
source_last_updated: 2026-06-17 (User Products) / 2026 (Preço por variação, Multi-origin stock) — ⚠ verify
volatile: true — highest-churn area of this Skill

Primary Mercado Livre architectural reference for: legacy vs User Products,
families, PARENT_PK / CHILD_PK, Items vs sale conditions, stock ownership and
distributed inventory. Other files cross-link here instead of restating.

## 1. Mental model — five distinct concepts, never collapsed

```
ProductMaster → Product Family → User Product → Item / sale condition → Inventory location
```

| Concept | What it is | Identity |
|---|---|---|
| **ProductMaster** | Product Factory's own source of truth — normalized facts, variants, evidence, marketplace-independent structure. **Not** an ML resource. | internal `product_id` |
| **internal Variant** | A Product Factory sellable unit (`PF-100-BLK`). Stable; never replaced by a marketplace id. | internal `variant_id` / SKU |
| **Product Family** | ML grouping of closely related User Products that share their family-defining values. **Not** a sellable inventory unit. | `family_id` (ML-assigned, **can change** — §14) |
| **User Product (UP)** | A seller's *sufficiently specific physical product / variant* (frame model X / black). Can be linked to **one or more** Items. | `user_product_id` (ML-assigned, not editable — §4) |
| **Item** | The marketplace publication / **sale condition** — *how* the product is offered (price, listing type, shipping, channel). | `item_id` (`MLB…`) |
| **legacy variation** | A `variations[]` entry inside one legacy Item. | `variation_id` |
| **Inventory location** | A physical stock allocation (`meli_facility`, `seller_warehouse`, `selling_address`). | ML `store_id` / `network_node_id` |

> User Product = **WHAT** physical product this is. Item = **HOW** it is being
> offered. Stock location = **WHERE** the units are. Keep the three separate:
> price changes don't change product identity; stock-location changes don't
> change family identity.

## 2. Publication-model resolution — do this before building any payload

Resolve **before** constructing a publication payload or a stock write. Do **not**
infer the model from ProductMaster variant count (5 variants ≠ `variations[]`;
1 variant ≠ legacy single item).

| Result | Signals (OFFICIAL ⚠ verify — verified 2026-08-27 search-indexed) |
|---|---|
| `USER_PRODUCT` | seller has the **`user_product_seller`** tag on `GET /users/$USER_ID`; and/or the item has the **`user_product_listing`** tag or a non-null **`family_name`** / a `user_product_id`. |
| `LEGACY` | seller **not** activated for `user_product_seller`; item has `variations[]` / no `family_name`. |
| `UNRESOLVED` | seller tags / item context not yet read. → **REVIEW** (Correction 02A: pending, not `FAIL`). |

- Once `user_product_seller` activates, **new publications must use the User
  Products structure**; the seller could publish legacy only *until* activation.
  Rollout is in waves by category / seller profile. (OFFICIAL ⚠ verify)
- After activation ML runs a **unification** of the seller's mono-variant /
  non-variant items, grouping them under `user_product_id`s and giving them
  `family_name`. Existing legacy items **coexist**; they migrate only at the
  seller's request (front-end or integrator) — never assume an automatic mass
  rewrite (§14).

## 3. LEGACY VARIATIONS  (`variations[]`) — OFFICIAL, LEGACY MODEL ONLY (⚠ verify)

Applies only when model resolution = `LEGACY`. Never let these rules leak into a
User Products payload.

- One parent Item holds `title`, `price`, `description`, base pictures.
- Each `variations[]` entry: `attribute_combinations` (the axis values, e.g.
  `Cor = Azul`), `available_quantity` (per-variant stock — valid **here**),
  `price` (generally inherits the parent — no true per-variant pricing; §9),
  `picture_ids`, and the **`SELLER_SKU`** attribute for the SKU.
- Only attributes the category marks as variation attributes
  (`allow_variations` / `variation_attribute`) may be axes.
- **Max variations per item** (OFFICIAL, LEGACY — ⚠ verify, "Variations" doc,
  stated 2022-12-14): **100** for standard categories, **250** for Fashion,
  Mobile Accessories and Auto Parts. This is a **legacy-model item limit**, not a
  global Product Factory variant cap.

## 4. USER PRODUCTS — core model (OFFICIAL ⚠ verify — verified 2026-08-27 search-indexed)

- A **User Product** represents the seller's specific physical product / variant;
  ML assigns `user_product_id` (not editable — persist it).
- **One UP → one or more Items.** Each Item is a *sale condition* for the same
  physical product (e.g. red iPhone UP → item1 with 3 installments, item2 at a
  different price). Do **not** treat one Item as synonymous with one physical
  product.
- Every UP belongs to a **family** (`family_id`); a family groups several UPs and
  displays them as pickers on the User Product Page.
- **Item count per UP** — reported cap **30** sale conditions / Items per User
  Product; exceeding it errors. Treat the number as DYNAMIC / ⚠ verify; report
  whether it is global or context-specific once confirmed.
- **New-model publication does not use `variations[]`.** Sending `variations[]`
  for a resolved `USER_PRODUCT` seller/context is a **BLOCKER**.
- **Title** is **auto-generated** by ML from product info in the new model — the
  seller does not send `title`. A resolved attempt to send a forbidden manual
  `title` → **BLOCKER → FAIL**; model still `UNRESOLVED` → **REVIEW**
  (Correction 02A). `family_name` semantics: `titles-and-family-name.md` (owned by
  Correction 04 — not re-defined here).

## 5. Families

- `family_name` is the shared family identity — same across every UP in the
  family, never a picker value (`titles-and-family-name.md`). Length ≤ the
  domain's `max_title_length`. It is **not related to sale conditions** (price,
  listing type). It **can be updated** and then syncs across the family (§13).
- Membership: `GET /sites/$SITE_ID/user-products-families/$FAMILY_ID` returns all
  UPs in a family. There is no documented "list all my families" endpoint —
  reach families via their UPs / Items.
- Family membership must be **evidence-backed** — do not group two UPs just
  because they share a brand, similar titles/images, or were imported together;
  and do not split legitimate variants into unrelated families without reason.

## 6. Family calculation — PARENT_PK / CHILD_PK (OFFICIAL ⚠ verify — verified 2026-08-27 search-indexed)

Family grouping is **richer than "same `family_name` = same family"**. It
considers `family_name` / name, domain, seller/user, condition, and the PK tags:

| Tag | Role in the family calculation |
|---|---|
| **`PARENT_PK`** | Values **must be identical** across every UP in the family. These define the family (e.g. `BRAND`, `MODEL` — *if the domain marks them PARENT_PK*). |
| **`CHILD_PK`** | Only the attribute **id + name** contribute; the **value may vary** between UPs in the family (the picker dimension, e.g. `COLOR` — *if the domain marks it CHILD_PK*). |
| **Custom attributes** | Like CHILD_PK — id + name contribute; value may vary. Do not invent a marketplace variation axis just because ProductMaster has an internal one. |
| **`read_only` PK** | A PARENT_PK or CHILD_PK attribute tagged `read_only` is **not considered** in the family calculation. |

- **Never hardcode** which attributes are PARENT_PK vs CHILD_PK — resolve it from
  the domain/category attribute metadata (`categories.md`). `BRAND`/`MODEL` are
  *not* always PARENT_PK; `COLOR`/`SIZE` are *not* always CHILD_PK.
- Resolved PARENT_PK conflict among UPs that Product Factory is forcing into one
  family → **BLOCKER**. A required CHILD_PK axis with a missing, non-evidence-
  backed value on a sellable UP → **BLOCKER**; the requirement still pending
  marketplace metadata → **REVIEW** (Correction 02A).

## 7. ProductMaster → Mercado Livre mapping

Keep internal identity stable; marketplace ids are **external mappings**, not
Product Factory identity (INTERNAL architecture):

```
internal variant_id / SKU
      ↓  (persist the mapping)
MercadoLivreIdentity {
   user_product_id      (new model)
   family_id            (new model — may change, §14)
   item_ids[]           (one UP → many Items)
   legacy_variation_id? (legacy model)
}
```

- Never replace an internal `variant_id` with `item_id`, `user_product_id` or
  `variation_id`. Persist every external id a publication returns
  (`item_id`, `user_product_id`, `family_id`, stock-location mappings) — do not
  rediscover identity from titles later.
- Identify variants by a stable internal id, never by a mutable display string
  (`"Black / Large"`).
- Axis mapping is not automatic: `ProductMaster axis → resolve marketplace
  attribute → PARENT_PK? CHILD_PK? normal? custom? unsupported?` No safe mapping
  → **REVIEW**; an executed marketplace check proving the structure invalid →
  **FAIL**.
- Before assuming a **new** UP will be created: ML may associate an Item with an
  **existing** UP when product identity matches. Don't promise deterministic UP
  creation — persist whatever `user_product_id` the marketplace returns.

## 8. SKU & GTIN placement

- **SKU**: the ML-recognised field is the **`SELLER_SKU`** attribute (on the item,
  and on variation attributes in the legacy model). **`seller_custom_field`** is
  the seller's private field, unrelated, and not used by ML for identification /
  orders — do **not** treat it as the marketplace SKU. Product Factory keeps its
  own stable SKU per sellable unit; never invent one.
- **GTIN** (evidence rules: Correction 03 — not re-defined here): a barcode
  belongs to the **specific sellable unit** (variant / UP). Do not propagate one
  GTIN across all variants unless evidence proves it applies to all. Legacy
  placement is per-`variations[]` attributes; the new model places it on the UP
  identity — confirm current field mechanics (⚠ verify).

## 9. Price model

| Model | Where price lives |
|---|---|
| **LEGACY** | Inside one Item with `variations[]`. Historically the payload could carry per-variation price fields, but display/payment required a common price and current APIs may reject differing prices — treat any exact per-variation-price rule as **LEGACY ONLY** and re-verify. |
| **USER PRODUCTS** | Price is an **Item / sale-condition** concern. Different Items linked to the same UP may carry different price / listing type / shipping. Price is **not** a UP field. `family_name` is unrelated to price. |

Do not couple price to identity: a price change must not trigger a ProductMaster
variant-identity change. (Business pricing/margin logic is out of scope —
`pricing-and-commercial.md`.)

## 10. Inventory model — ownership

- **New model**: stock belongs to the physical **User Product**, distributed
  across **stock locations** — not to an arbitrary commercial Item, and not a
  single `available_quantity` forever.

  ```
  User Product → Stock → { locations… }
  ```

- **Legacy model**: `available_quantity` on the Item / `variations[]` entry
  remains valid.
- Product Factory keeps its **own** inventory source of truth keyed by internal
  variant / SKU; the marketplace is a projection / execution target, never the
  master stock (INTERNAL architecture). Inventory identity must not be coupled to
  ML-specific ids (other marketplaces map from the same internal SKU).

## 11. `available_quantity` — CRITICAL, context-dependent

- **Do NOT state "update `/items` `available_quantity` to synchronise stock" as a
  universal rule.**
- For **multi-origin / multiwarehouse** accounts, `available_quantity` **must not
  be used** as the stock-write mechanism on `/items` — it may be ignored,
  rejected, or derived from User Product stock. Use the stock-location operations
  (§12). (OFFICIAL ⚠ verify — verified 2026-08-27 search-indexed.)
- For **legacy** accounts it remains the write mechanism.
- Treat Item-level `available_quantity` as **derived / aggregated / non-writable**
  wherever multi-origin applies. This is why model + stock-mode detection is
  mandatory.

## 12. Multi-origin / multiwarehouse (OFFICIAL ⚠ verify — verified 2026-08-27 search-indexed; largely AR/CL docs, MLB support DYNAMIC)

**Stock-mode resolution** (before any stock write):
`LEGACY ITEM STOCK` / `USER PRODUCT STOCK` / `MULTI-ORIGIN (multiwarehouse)` /
`UNRESOLVED` → REVIEW.

- Seller capability tag: **`warehouse_management`** on `GET /users/$USER_ID`.
  For an activated multiwarehouse seller: publishing goes through
  **`POST /items/multiwarehouse`**, the payload carries **`stock_locations`**
  (`store_id`, `network_node_id`, `quantity`), `available_quantity` is invalid,
  and the response returns a **`user_product_id`** that must be persisted for
  later stock ops.
- **Stock-location types** (a UP may hold up to two: `(selling_address` +
  `meli_facility)` **or** `(seller_warehouse` + `meli_facility)`):
  - **`meli_facility`** — Mercado Livre-managed Fulfillment (Full) stock.
    Seller API writes to this location are generally **not allowed** — never try
    to overwrite Full stock directly.
  - **`seller_warehouse`** — seller-managed; the main multi-origin location type.
    Write: `PUT /user-products/{user_product_id}/stock/type/seller_warehouse`.
  - **`selling_address`** — seller origin for non-Fulfillment logistics;
    **site-dependent** (documented for AR/CL — do not assume MLB support). DYNAMIC.
- **Read stock**: `GET /user-products/{user_product_id}/stock` → per-location
  `type`, `network_node_id`, `store_id`, `quantity`. Do not treat Item
  `available_quantity` as location-level truth.
- **Discover locations**: `GET /users/{user_id}/stores/search?tags=stock_location`
  → the seller's stores, each with `store_id` + `network_node_id`. Location ids
  **must come from this API** — never invent them or derive them from internal
  warehouse names. Product Factory maps its own warehouse identity → ML
  `store_id` / `network_node_id`.
- **`stock-locations not found`** does **not** mean `quantity = 0`. It usually
  means the stock locations were not initialised / associated to the UP yet — or
  the wrong UP / wrong mode / unsupported context. Location creation and
  association must happen before a stock write.
- **Full + Flex / distributed**: one UP may hold stock across more than one
  location — do not assume "one User Product = one warehouse".

## 13. Synchronization (OFFICIAL ⚠ verify — verified 2026-08-27 search-indexed)

- A `PUT /items/{id}` touching **User-Product-level** fields (shared/product
  attributes, `family_name`, …) is **asynchronously replicated by ML across all
  Items linked to the same `user_product_id`**. `family_name` updates sync across
  the family.
- **Commercial / sale-condition** fields (price, listing type, shipping) stay
  Item-specific and are **not** propagated.
- Async ⇒ a `PUT` accepted ≠ every associated Item is immediately consistent.
  Integration must accommodate: *request accepted → propagation pending →
  re-fetch / notification / verify → confirmed*. (Do not build workers/queues
  here — record the operational implication only.)
- **Family editor**: `POST /sites/$SITE_ID/user-products-families/{family_id}/tasks`
  (⚠ verify path) edits common family content asynchronously with a task status.
  Document as an execution mechanism; not required in every workflow.
- **Stock-location change notifications**: may be partial-rollout / "coming soon"
  — ⚠ verify; do not design required architecture around an unavailable topic.

## 14. Migration & coexistence

Possible states (procedural): `LEGACY_EXISTING`, `UP_EXISTING`, `UP_NEW`,
`MIGRATION_IN_PROGRESS`, `UNRESOLVED`.

- Legacy Item structures and new User Product structures **coexist** for the same
  seller. Read current marketplace state first; do not recommend rewriting every
  legacy listing just because the seller is activated.
- `family_id` is **not** a permanently immutable business identifier — changing
  `family_name` / brand / model / a family-defining attribute can move a UP to a
  **different `family_id`**. Product Factory's internal product/variant ids must
  stay stable regardless.
- `user_product_id` is ML-assigned and not editable — it is an external mapping,
  never Product Factory's cross-marketplace identity.
- **UPtin**: a migration path where ML generates `family_name` itself
  (Correction 04). Do not derive the general User Products flow from UPtin
  behaviour; ⚠ verify specifics.

## 15. Catalog coexistence

A **Catalog Product** (`catalog_product_id`, marketplace catalog identity) is
**not** the same as a **User Product** (`user_product_id`, seller/product
identity). `catalog_product_id ≠ user_product_id`; neither replaces the other. An
Item may participate in both systems. Full catalog rules: `catalog.md`.

## 16. Quality checks & blockers

`VARIANTS` dimension + the `PUBLICATION_MODEL` / `INVENTORY_MODE` procedural
checks (`quality-audit.md`).

**BLOCKER → FAIL** (executed / resolved incompatibility):
- `variations[]` sent for a resolved `USER_PRODUCT` seller/context.
- Manual `title` sent where the new-model flow generates it.
- `/items` `available_quantity` written for a resolved multi-origin
  (`warehouse_management`) account.
- Stock assigned to a location that is not the seller's; an invented
  `store_id` / `network_node_id`.
- Using another variant's `user_product_id`; mixing Items from different UPs.
- A `CHILD_PK` value sent as shared family identity; invented PARENT_PK / CHILD_PK
  mappings; a resolved PARENT_PK conflict forced into one family.
- A required `CHILD_PK` variant value missing after an executed validation;
  invented colour / size / capacity.
- Exceeding the resolved per-UP Item cap or the legacy variation limit.

**REVIEW** (pending / dynamic):
- Seller model or stock mode `UNRESOLVED`.
- PK metadata unavailable; a required CHILD_PK requirement pending.
- Warehouse / location mapping unavailable; site support for a location type
  unknown.
- Legacy-vs-UP status uncertain.

Content readiness is separate: a ProductMaster can be **content-ready** while
stock publication is not yet executable (Correction 02 layers; Correction 07 owns
the dual-status normalization).

## 17. Dynamic checks (Correction 02A semantics — pending → REVIEW; executed-unmet → FAIL)

seller tags/capabilities (`user_product_seller`, `warehouse_management`);
publication model; item model; PK metadata & `read_only` flags; legacy max-
variation limits; per-UP Item cap; User Product availability for the
site/category; stock locations & their ids; site support for `selling_address` /
other location types; current family/UP relationships; whether `available_quantity`
is writable for this account.

## Sources

- User Products — https://developers.mercadolivre.com.br/pt_br/user-products , https://developers.mercadolibre.com.ar/en_us/user-products — Developers — updated 2026-06-17 (per requester); verified 2026-08-27 (search-indexed; live 403) — UP = specific physical product, `user_product_id` ML-assigned, 1 UP → many Items (sale conditions), family/`family_id`, `user_product_seller` / `user_product_listing` tags, post-activation unification, auto-generated title, async replication of product-level PUTs, `GET /users/$SELLER_ID/items/search?user_product_id=`, `GET /sites/$SITE/user-products-families/$FAMILY_ID`.
- Family calculation (PARENT_PK / CHILD_PK / custom / read_only) — Developers "User Products" — verified 2026-08-27 (search-indexed; live 403) — PARENT_PK values identical across family; CHILD_PK & custom contribute id+name only; `read_only` PK excluded; family calc also weighs name/domain/seller/condition.
- Preço por variação / Price per variation — https://developers.mercadolivre.com.br/pt_br/preco-variacao — Developers — 2026 ⚠ verify — consulted 2026-08-27 — per-Item price/shipping/stock, ~30 sale conditions per UP, rollout in waves, seller-request migration only.
- Variations (legacy) — https://developers.mercadolivre.com.br/pt_br/variacoes — Developers — verified 2026-08-27 (search-indexed; live 403) — `variations[]` / `attribute_combinations` / `SELLER_SKU` / `picture_ids`; max **100** variations per item (**250** Fashion / Mobile Accessories / Auto Parts), stated 2022-12-14.
- Multi-Origin Stock / Gestión de stock multiorigen / User Products — https://developers.mercadolibre.com.ar/en_us/multi-origin-stock , https://developers.mercadolibre.com.ar/stock-multiwarehouse — Developers (AR — MLB support DYNAMIC) — verified 2026-08-27 (search-indexed; live 403) — `warehouse_management` tag, `POST /items/multiwarehouse` + `stock_locations` (`store_id`/`network_node_id`/`quantity`), `available_quantity` invalid in multi-origin, `GET/PUT /user-products/{id}/stock[/type/seller_warehouse]`, `GET /users/{id}/stores/search?tags=stock_location`, location types `meli_facility` (ML-managed, no seller write) / `seller_warehouse` / `selling_address` (site-dependent), UP holds up to two typologies, `stock-locations not found` = not initialised (≠ qty 0), Full+Flex coexistence.
- SELLER_SKU vs seller_custom_field — Developers "Variations / Items & Searches" — verified 2026-08-27 (search-indexed; live 403) — `SELLER_SKU` is the ML-recognised SKU attribute; `seller_custom_field` is seller-internal only, unrelated.
- Family editor `POST /sites/$SITE/user-products-families/{family_id}/tasks` — Developers — ⚠ verify (path/behaviour not directly confirmed this pass).
