# Product / listing structure — Shop / Item / Tier Variation / Model

last_reviewed: 2026-08-28
phase_02_3_reviewed: 2026-09-03
phase_02_2_reviewed: 2026-08-30
volatile: true
classification: OFFICIAL (structure) — entity spine PRIMARY_VERIFIED (SPD-010/011/015/020); detail not covered by a primary page stays `⚠ verify`
phase_02_3_note: >-
  PRIMARY (SPD-010, SPD-011, SPD-015, SPD-020). Entity spine Shop → Item → Tier
  Variation → Model is **PRIMARY_VERIFIED**. `add_item` returns `item_id`
  (int64). `get_item_base_info` takes **`item_id_list` limit [0,50]** — no
  `product_id` request parameter exists in the primary corpus (drop the
  cross-border-alias worry). NEW: `get_item_base_info` returns **`ssp_id` =
  "Shopee Standard Product"** id — a catalogue-like node (relates to gap G6) —
  and `tag.kit` (bool). **`model_id = 0` = "no model item"**; Shopee keeps an
  internal **default model** for no-variation items (no separate hidden id is
  exposed). `item_status` ∈ `NORMAL` / `BANNED` / `UNLIST` / `SELLER_DELETE` /
  `SHOPEE_DELETE` / `REVIEWING`. Persist `shop_id` / `item_id` / `model_id` /
  `brand_id` / `image_id` / `video_upload_id`; write internal SKU into `item_sku`
  / `model_sku` (≤ 100 chars).
phase_02_2_note: >-
  HISTORICAL (superseded by phase_02_3_note above). Entity spine Shop → Item →
  Tier Variation → Model was then a `SEARCH_INDEXED` candidate; Phase 02.3
  (SPD-010/011/015/020) made it `PRIMARY_VERIFIED`. `get_item_base_info.item_id_list`
  limit is now confirmed **[0, 50]**. No `product_id` request parameter exists in
  the primary corpus (`PRIMARY_NOT_FOUND` as a local param). Models are created
  by a separate call (`init_tier_variation` / `add_model`) after `add_item`. See
  `research/shopee-api-contract/phase-02.2-report.md` §8–§10, §13.

## 1. The entity model (current best understanding)

```
Shop  (shop_id)                                   — the seller storefront, region BR
 └── Item  (item_id)                              — THE LISTING
      │     item_sku    seller-set (parent SKU)   — seller-side listing identity
      │     item_status NORMAL | BANNED | UNLIST | SELLER_DELETE | SHOPEE_DELETE | REVIEWING
      ├── Tier Variation                          — the variation axes
      │     • 0, 1, or 2 tiers — max 2 is PRIMARY_VERIFIED (SPD-015)
      │     • positional — no id; an ordered option_list per tier
      │     • tier-1 options may each carry one image
      └── Model  (model_id)                       — THE SELLABLE VARIANT
            model_sku   seller-set, mutable
            tier_index[] which option of each tier this model is (e.g. [0,2])
            price, stock per model
```

For an item with **no** variation, per-model price/stock operations use the
**`model_id = 0` "no model item" addressing convention** (`PRIMARY_VERIFIED`,
SPD-016/027/028). Whether Shopee exposes a fully queryable default-model *entity*
for every such item across every API is inferred from a single FBS-context phrase
("only has a default model") and is **`PRIMARY_PARTIAL`** — do not overstate it.
Product Factory maps `variant_id → (item_id, model_id = 0)` and keeps its own
sellable-unit identity either way.

Terminology (Phase 02.3, `PRIMARY_VERIFIED`): **the id Product Factory persists
is `item_id`** — Shopee-assigned by `add_item` (int64). `get_item_base_info`
takes `item_id_list`, limit **[0, 50]** per call (SPD-011). **No `product_id`
request parameter exists in the primary corpus** — the integrator-page
`product_id` is `PRIMARY_NOT_FOUND` as a local param and is **not** a second
local listing identity. `get_item_base_info` also returns **`ssp_id`** (Shopee
Standard Product, a catalogue-like node — SKILL.md gap G6). "model" is the v2
word; older v1 material says "variation".

Scope separation:
- `v2.product.*` — local marketplace listing / product operations (the BR
  listing scope this Skill targets) — **corroborated API contract candidate**.
- `v2.global_product.*` — cross-border / global-product domain — **out of the
  current Product Factory Shopee Brazil listing scope** unless later required.
  Global-product entities and ids must not leak into the local Item / Model
  mapping.

## 2. Entity confidence table

| Entity | Meaning | Shopee id | Assigned by | Stable? | Buyer-visible? | Product Factory mapping | Verification |
|---|---|---|---|---|---|---|---|
| Shop | seller storefront on Shopee BR | `shop_id` | Shopee | yes | no | account / shop mapping | `SEARCH_INDEXED` |
| Item | the listing | `item_id` (int64) | Shopee (on create) | yes (life of listing) | in URL | 1 ProductMaster → 1 Item per shop | `PRIMARY_VERIFIED` (SPD-010) |
| Tier Variation | the item's variation axes | none (positional) | seller-defined | 0–2 tiers; structure change LOCKED under promotion | option names yes | internal variation axes | `PRIMARY_VERIFIED` (SPD-015); name-length caps DYNAMIC |
| Model | one sellable combination of tier options | `model_id` (+ `tier_index[]`); `model_id = 0` = no-model | Shopee (on `add_model` / `init_tier_variation`) | yes (life of model) | no | internal `variant_id` / SKU ↔ `model_id` | `PRIMARY_VERIFIED` (SPD-015/020) |
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

## 4. Item-level fields

The `add_item` required / optional field set is `PRIMARY_VERIFIED` (SPD-010) —
see `references/api-and-auth.md` §0 and claim-registry `SCL-021` / `SCL-022` for
the authoritative list. In outline: `item_name`, `description`
(+ `description_type` `normal` | `extended` — extended = whitelist), `category_id`,
`brand` (`{ brand_id, original_brand_name }` — always required),
`attribute_list` (must cover all `mandatory`), `image` (`{ image_id_list }`),
`video_upload_id` (one), `weight` (KG, required), `dimension`
(`{ package_length, package_width, package_height }` cm), `logistic_info`
(`{ logistic_id, enabled, … }`), `condition` (`NEW` | `USED`),
`pre_order` (`{ is_pre_order, days_to_ship }`), `item_sku`, `item_status`
(`UNLIST` | `NORMAL`), `seller_stock`, `original_price`, `gtin_code`
(model-level; BR + TW), `tax_info` + `export_cfop` (BR), `wholesale` tiers,
`item_dangerous` (ID/MY only). `update_item`'s full field list is
`PRIMARY_NOT_FOUND` — treat it as `⚠ verify`.

## 5. Listing lifecycle

`item_status` enum (**`PRIMARY_VERIFIED`**, SPD-011): `NORMAL` (live), `UNLIST`
(hidden / not for sale — seller-toggled), `BANNED` (removed by Shopee for a
violation), `SELLER_DELETE` / `SHOPEE_DELETE` (deleted by seller / by Shopee —
distinct states; there is **no** plain `DELETED`), `REVIEWING` (pending
moderation). `add_item` accepts `UNLIST` | `NORMAL`; a draft via API =
`UNLIST` + `scheduled_publish_time` (SPD-010).

State model: `NOT_CREATED → (create) → [REVIEWING] → NORMAL ⇄ UNLIST`, with
`BANNED` / `SELLER_DELETE` / `SHOPEE_DELETE` terminal-ish. Whether an edit
re-triggers `REVIEWING` is **`PRIMARY_NOT_FOUND`** (not stated in the corpus).
Full graph and enforcement in `references/moderation-and-enforcement.md`.

## 6. NOT part of this model (Mercado Livre concepts — do not import)

`User Product`, `Family` / `family_name`, `PARENT_PK` / `CHILD_PK`,
`Multi Origem` (`STANDARD` / `MULTI_ORIGIN_*`), `listing_allowed`,
`EMPTY_GTIN_REASON`, shared catalog product page / Buy Box,
`POST /items/validate`, `/performance`, `moderations/last_moderation`.

The model **is** the sellable unit and belongs to one item — there is no
ML-style "what is sold" entity separate from the listing. Tier axes are
seller-declared and positional, not derived from attribute metadata tags.
Whether Shopee BR has a full catalogue / "produto" grouping concept is only
**partially** resolved (SKILL.md gap G6 — `get_item_base_info` returns `ssp_id`,
a catalogue-like node; no Buy-Box evidence) — assume every listing is standalone.

## Sources

- Entity spine + lifecycle order + `item_id` naming (Phase 02.2) —
  `github.com/QuoVadis86/shopee-sdk`, `github.com/congminh1254/shopee-sdk`
  (`docs/managers/product.md`), `rollout.com` — community SDK / external —
  consulted 2026-08-28 — `SEARCH_INDEXED` · MEDIUM · MULTI_SOURCE (non-primary
  contract candidate); `phase-02.2-report.md` §8–§10, §13, §29 (C7).
  (Phase 02.3 superseded these: no `product_id` request param exists in the
  corpus; `item_id_list` limit confirmed **[0, 50]**; `item_status` enum
  `PRIMARY_VERIFIED` — SPD-011.)
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
