# Product / listing structure — Shop / Item / Tier Variation / Model

last_reviewed: 2026-08-28
phase_02_2_reviewed: 2026-08-30
volatile: true
classification: OFFICIAL (structure) — verification SEARCH_INDEXED; every API-contract detail `⚠ verify`
phase_02_2_note: >-
  Entity spine Shop → Item → Tier Variation → Model corroborated across two
  independent SDKs (PARTIALLY_CONFIRMED). The persisted listing id is `item_id`;
  some read endpoints accept it as `item_id_list` (≤ 50) or `product_id`. Models
  are created by a **separate call** (`init_tier_variation` / `add_model`) after
  `add_item`. See `research/shopee-api-contract/phase-02.2-report.md` §8–§10.

## 1. The entity model (current best understanding)

```
Shop  (shop_id)                                   — the seller storefront, region BR
 └── Item  (item_id)                              — THE LISTING
      │     item_sku    seller-set, mutable       — seller-side listing identity
      │     item_status NORMAL | UNLIST | BANNED | DELETED  (REVIEWING? — `⚠ verify`)
      ├── Tier Variation                          — the variation axes
      │     • ≤ 2 tiers (corroborated across SDKs; cap `⚠ verify` for BR)
      │     • positional — no id; an ordered option_list per tier
      │     • tier-1 options may each carry one image
      └── Model  (model_id)                       — THE SELLABLE VARIANT
            model_sku   seller-set, mutable
            tier_index[] which option of each tier this model is (e.g. [0,2])
            price, stock per model
```

An item with **no** variation has no models; its single sellable unit is the
item itself (`item_id` + `item_sku`).

Terminology (Phase 02.2, `SEARCH_INDEXED` MEDIUM): v2 mixes "item" and
"product". **The id Product Factory persists is `item_id`** — Shopee-assigned by
`add_item`. Read endpoints take it as `item_id_list` (≤ 50 per call, per S13) or,
in some integrator docs, as `product_id` (S14). `item = product = the listing
entity`. "model" is the v2 word; older v1 material says "variation". A separate
`v2.global_product.*` family exists for cross-border sellers — **out of scope**
here; BR listings are `v2.product.*`.

## 2. Entity confidence table

| Entity | Meaning | Shopee id | Assigned by | Stable? | Buyer-visible? | Product Factory mapping | Verification |
|---|---|---|---|---|---|---|---|
| Shop | seller storefront on Shopee BR | `shop_id` | Shopee | yes | no | account / shop mapping | `SEARCH_INDEXED` |
| Item | the listing | `item_id` | Shopee (on create) | yes (life of listing) | in URL | 1 ProductMaster → 1 Item per shop | `SEARCH_INDEXED`; contract `⚠ verify` |
| Tier Variation | the item's variation axes | none (positional) | seller-defined | mutable pre-sales; restricted after | option names yes | internal variation axes | `SEARCH_INDEXED`; caps `⚠ verify` |
| Model | one sellable combination of tier options | `model_id` (+ `tier_index[]`) | Shopee (on `add_model` / `init_tier_variation`) | yes (life of model) | no | internal `variant_id` / SKU ↔ `model_id` | `SEARCH_INDEXED`; contract `⚠ verify` |
| `item_sku` / `model_sku` | seller-controlled identity string | value only | seller / Product Factory | mutable | mostly no (`⚠ verify`) | store the internal SKU here | `SEARCH_INDEXED` |
| Brand | brand attribute value | `brand_id` | Shopee / seller+approval | stable | yes (as name) | — | `SEARCH_INDEXED` |
| Category | leaf category | `category_id` | Shopee | stable-ish (tree changes) | breadcrumb | 1 Item → 1 leaf | `SEARCH_INDEXED` |
| Media | image / video asset | `image_id`, `video_upload_id` | Shopee | id stable; **CDN URL expires** | yes | persist the **id**, not the URL | `SEARCH_INDEXED` |
| `merchant_id` | groups shops for SIP / cross-border | `merchant_id` | Shopee | stable | no | — | `SEARCH_INDEXED` |

## 3. Identifier & mapping discipline (`SHARED_CORE_CANDIDATE`, INTERNAL design)

- Product Factory persists its own stable `variant_id` / SKU. Shopee ids are
  **external mappings** stored against it: `variant_id ↔ (shop_id, item_id,
  model_id)`; the internal SKU is written **into** `item_sku` / `model_sku`.
- Persist media ids (`image_id`, `video_upload_id`) — **not** CDN URLs, which
  expire.
- SKU appears to be **optional to publish** and **not uniqueness-enforced by
  Shopee** (`⚠ verify`). Product Factory enforces its own SKU uniqueness; never
  rely on Shopee to dedupe.
- Never use `item_name` or tier-option text as canonical identity when
  `item_id` / `model_id` / SKU exist.
- A Shopee listing is a **projection** of the product. It is never a source of
  `ProductMaster` truth.

## 4. Item-level fields (reconstructed — names `SEARCH_INDEXED`, requiredness `UNVERIFIED`)

`item_name`, `description` (+ `description_type` `normal` | `extended`),
`category_id`, `brand` (`{ brand_id, original_brand_name }`), `attribute_list`,
`image` (`{ image_id_list }`), `video_upload_id`, `weight` (kg), `dimension`
(`{ package_length, package_width, package_height }` cm), `logistic_info`,
`condition` (`NEW` | `USED`), `pre_order` (`{ is_pre_order, days_to_ship }`),
`item_sku`, `item_status`, stock / `seller_stock`, `price`, `tax_info` / fiscal
fields (BR — `UNVERIFIED`), `wholesale` tiers, dangerous-goods flag
(`UNVERIFIED`). Treat every field name as provisional until the portal is read.

## 5. Listing lifecycle

`item_status` — corroborated set: `NORMAL` (live), `UNLIST` (hidden / not for
sale — seller-toggled), `BANNED` (removed by Shopee for a violation), `DELETED`
(removed by seller or system). A `REVIEWING` / under-review state appears in some
responses — `UNVERIFIED`. Draft-via-API — `UNVERIFIED`.

Provisional state model: `NOT_CREATED → (create) → [REVIEWING?] → NORMAL ⇄
UNLIST`, with `BANNED` / `DELETED` as terminal-ish. Whether edits re-trigger
review — `⚠ verify`. Full graph and enforcement in
`references/moderation-and-enforcement.md`.

## 6. NOT part of this model (Mercado Livre concepts — do not import)

`User Product`, `Family` / `family_name`, `PARENT_PK` / `CHILD_PK`,
`Multi Origem` (`STANDARD` / `MULTI_ORIGIN_*`), `listing_allowed`,
`EMPTY_GTIN_REASON`, shared catalog product page / Buy Box,
`POST /items/validate`, `/performance`, `moderations/last_moderation`.

The model **is** the sellable unit and belongs to one item — there is no
ML-style "what is sold" entity separate from the listing. Tier axes are
seller-declared and positional, not derived from attribute metadata tags.
Whether Shopee BR has any catalogue / "produto" grouping concept is
**UNRESOLVED** (SKILL.md gap G6) — assume every listing is standalone.

## Sources

- Entity spine + lifecycle order + `item_id`/`product_id` naming (Phase 02.2) —
  `github.com/QuoVadis86/shopee-sdk`, `github.com/congminh1254/shopee-sdk`
  (`docs/managers/product.md`), `rollout.com` — community SDK / external —
  consulted 2026-08-28 — `SEARCH_INDEXED`, MEDIUM; `phase-02.2-report.md` §8–§10,
  §29 (C7). `item_status` enum values **not** re-verified this phase — still
  `STILL_UNVERIFIED`.
- v2 endpoint names & `item_status` enum — `github.com/wjp-letgo/shopeego` —
  community SDK — consulted 2026-08-28 — `SEARCH_INDEXED`.
- v1 field names, `tier_index` combination semantics — `github.com/teacat/shopeego`
  — community SDK — consulted 2026-08-28 — `SEARCH_INDEXED` (v1; do not trust
  numbers).
- v2 hosts, "item" vs "product" naming, "store image ids not URLs" —
  `developer.inlinex.com.sg`, `rollout.com`, `api2cart.com` — external —
  consulted 2026-08-28 — `SEARCH_INDEXED`.
- ML non-mappings — `.claude/skills/mercado-livre-listing-best-practices/` —
  internal — architectural reference only.
- Full detail: `research/shopee-listing-skill/discovery-report.md` §3, §7,
  §17, §26–§27, §"Entity Table".
