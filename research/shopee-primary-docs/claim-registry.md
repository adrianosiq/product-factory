# Claim Registry — Phase 02.3

last_updated: 2026-09-03 (Phase 02.3 primary extraction)
status: **Reconciled against the 35-PDF primary corpus.** Most claims are now
`PRIMARY_VERIFIED` / `CORRECTED` / `PRIMARY_PARTIAL`; a few are
`PRIMARY_NOT_FOUND_IN_PRIMARY_CORPUS` (logistics service, media_space,
content-diagnosis). Evidence column = supporting `SPD-xxx`. See
`extraction-report.md` and `reconciliation-report.md`.

One row per implementation-critical claim Product Factory intends to depend on.
Seeded from the Phase 02.2 report (`research/shopee-api-contract/phase-02.2-report.md`)
and Correction 02.2A. `New state` moves **per claim, individually** — never
bulk-upgrade because an endpoint's page arrived.

State vocabulary:
`SEARCH_INDEXED` · `UNVERIFIED` · `STILL_UNVERIFIED` (checked against ingested
primary docs, not answered) · `PRIMARY_VERIFIED` (an ingested primary artifact
directly supports it) · `CORRECTED` · `CONFLICTING`.
Scope: `GLOBAL_API` · `BRAZIL` · `OTHER_MARKET` · `UNKNOWN_MARKET_SCOPE`.

## Auth

| Claim ID | Claim | Previous state | Evidence | New state | Scope |
|---|---|---|---|---|---|
| SCL-001 | Auth is OAuth per shop: `partner_id`+`partner_key` → redirect → `code` → `access_token` + `refresh_token` | `SEARCH_INDEXED` · MEDIUM · MULTI_SOURCE | SPD-001 | PRIMARY_VERIFIED — OAuth per shop/main-account; `code`→`token/get`→`access_token`+`refresh_token`; refresh via `access_token/get` | `GLOBAL_API` |
| SCL-002 | `access_token` lifetime ≈ 4 h; `refresh_token` ≈ 30 d, rotates | `SEARCH_INDEXED` | SPD-001 | PRIMARY_VERIFIED — access_token 4 h; refresh_token 30 d (single-use per shop/merchant); code 10 min; timestamp 5 min; auth validity ≤360 d (SPD-006 says 365 — minor CONFLICT) | `GLOBAL_API` |
| SCL-003 | Request signing = HMAC-SHA256 over `partner_id + api_path + timestamp + [access_token] + [shop_id]`; timestamp in seconds | `SEARCH_INDEXED`; **exact base string `UNVERIFIED`** | SPD-001 | PRIMARY_VERIFIED — Shop APIs: `partner_id+api_path+timestamp+access_token+shop_id`; Merchant: `…+merchant_id`; Public: `partner_id+api_path+timestamp`; HMAC-SHA256, key=partner_key, hex lowercase; timestamp in seconds | `GLOBAL_API` |
| SCL-004 | Version prefix is `/api/v2`; token endpoints `auth/token/get`, `auth/access_token/get` | `SEARCH_INDEXED` · MEDIUM | SPD-001,SPD-010 | PRIMARY_VERIFIED — `/api/v2`; `POST /api/v2/auth/token/get`, `POST /api/v2/auth/access_token/get` | `GLOBAL_API` |
| SCL-005 | Hosts: global `partner.shopeemobile.com`, sandbox `partner.test-stable.shopeemobile.com` | `SEARCH_INDEXED` | SPD-001,SPD-010 | PRIMARY_VERIFIED — Global `partner.shopeemobile.com`; CN `openplatform.shopee.cn`; Sandbox Global `openplatform.sandbox.test-stable.shopee.sg`. (Earlier `partner.test-stable.shopeemobile.com` CORRECTED) | `GLOBAL_API` |
| SCL-006 | `openplatform.shopee.com.br` is the canonical BR Product API base URL | `SEARCH_INDEXED` · LOW · SINGLE_SOURCE (existence signal only) | SPD-001,SPD-010..SPD-029 | PRIMARY_VERIFIED — every API-ref page lists Brazil host `https://openplatform.shopee.com.br/api/v2/<path>`; auth `https://open.shopee.com.br/auth` | `BRAZIL` |

## Brazil eligibility (P0)

| Claim ID | Claim | Previous state | Evidence | New state | Scope |
|---|---|---|---|---|---|
| SCL-010 | Shopee BR offers an Open Platform application/onboarding process for sellers/integrators | `SEARCH_INDEXED` (article titles 3445, 27314 — bodies unread) | SPD-002,SPD-003,SPD-004,SPD-008,SPD-009 | PRIMARY_VERIFIED (flow) / PRIMARY_DERIVED (Product API availability) — documented BR developer journey (login→dev profile→app→sandbox→Go Live→authorize); Console at `open.shopee.com/console`. "BR Product API is available" is a **derived** conclusion from BR hosts + BR `add_item` tax fields + BR journey, not one explicit Shopee sentence | `BRAZIL` |
| SCL-011 | Which seller/account types are eligible for BR Product API access | `UNRESOLVED` | SPD-004,SPD-035 | PRIMARY_VERIFIED (scoped) — the BR Developer Guide journey lists two developer types: `Registered Business Seller` and `Third-party Partner (ISV)` (SPD-004). SPD-035's BR SPI App guide separately names `Individual Seller` as a developer type in the Seller-Logistics / SPI context — a different scope, not a conflict | `BRAZIL` |
| SCL-012 | Whether BR Product API access requires approved-partner status / manual approval | `UNRESOLVED` (evidence leans approval-gated; **not** `CONFIRMED_RESTRICTED`) | SPD-003,SPD-004,SPD-005,SPD-009,SPD-035 | **PRIMARY_PARTIAL** — developer PROFILE is approval-gated (Shopee internal criteria — the criteria page is `PRIMARY_NOT_FOUND`); PRODUCTION is Go-Live-review-gated. **No separate per-category whitelist is documented** for the listing-API app categories (`Seller In-House System` / `ERP System`), unlike the SPI apps SPD-035 marks "Whitelist Only" — but absence of a documented whitelist is not proof none exists | `BRAZIL` |
| SCL-013 | Whether a BR sandbox/test environment is available | `UNRESOLVED` | SPD-008 | PRIMARY_VERIFIED — Sandbox is a separate environment, available before Go-Live; own account/shop/app/Partner Key; some features may not be implemented in Sandbox | `BRAZIL` |
| SCL-014 | Which API function-groups / permissions a BR partner app is granted | `UNRESOLVED` | SPD-005,SPD-007 | PRIMARY_VERIFIED — permissions = the App Category chosen at creation (immutable, 'no manual or extra permissions'); each API page lists its allowed APP types; wrong category → Permission Denied | `BRAZIL` |

## Item contract (P0)

| Claim ID | Claim | Previous state | Evidence | New state | Scope |
|---|---|---|---|---|---|
| SCL-020 | `add_item` creates a listing and returns `item_id` | `SEARCH_INDEXED` · MEDIUM · MULTI_SOURCE | SPD-010 | PRIMARY_VERIFIED — `POST /api/v2/product/add_item` → returns `item_id` int64 | `GLOBAL_API` |
| SCL-021 | `add_item` minimal required fields = `item_name`, `description`, `category_id`, `price`, `stock` | `SEARCH_INDEXED` (single SDK doc) | SPD-010 | PRIMARY_VERIFIED — required: `original_price`, `description`, `weight`, `item_name`, `category_id`, `image.image_id_list`, `logistic_info[].{enabled,logistic_id}`, `brand.{brand_id,original_brand_name}`, `pre_order.is_pre_order`; attributes must cover all `mandatory` | `GLOBAL_API` |
| SCL-022 | Full `add_item` required/conditional field set (images, weight, dimension, logistics, brand, attribute_list, condition, item_sku, …) | `UNVERIFIED` | SPD-010 | PRIMARY_VERIFIED (large) — optional set incl. `item_status`, `image_ratio` (3:4 whitelist), `condition` NEW/USED, `wholesale`, `video_upload_id` (one), `gtin_code`, `seller_stock`, `scheduled_publish_time` (UNLIST only, +1h..+90d), `size_chart_info`, `description_info` (extended = whitelist), `tax_info`+`export_cfop` (BR), `group_item_info`, `compatibility_info`, `certification_info` (PH), `item_dangerous` (ID/MY) | `GLOBAL_API` |
| SCL-023 | `add_item` / `update_item` response shape: `item_id`, warnings, failure list, `request_id`, field-level errors | `UNVERIFIED` | SPD-010 | PRIMARY_VERIFIED — response `item_id` + echo + `warning` + `request_id`; large enumerated business error-code set (title/desc/image/price/dts/category/brand/attribute/gtin/stock/warehouse). No structured `cause[]` array | `GLOBAL_API` |
| SCL-024 | `get_item_base_info` takes `item_id_list`, max 50 per call | `SEARCH_INDEXED` · SINGLE_SOURCE / provisional | SPD-011 | PRIMARY_VERIFIED — `get_item_base_info.item_id_list` limit **[0,50]** | `GLOBAL_API` |
| SCL-025 | `product_id` (seen on one integrator page) is a local alias for `item_id` — or a `v2.global_product` / cross-border concept | `UNRESOLVED` | SPD-011 | PRIMARY_NOT_FOUND (as local param) — no `product_id` request parameter in any of the 35 PDFs; NEW: `ssp_id` = Shopee Standard Product id (catalogue-like; see gap G6) | `UNKNOWN_MARKET_SCOPE` |
| SCL-026 | `item_sku` is seller-set, mutable, buyer-invisible | `UNVERIFIED` | SPD-011 | PRIMARY_PARTIAL — `item_sku` = seller-defined parent SKU (get_item_base_info: 'sometimes called parent SKU'); mutability/visibility not explicitly stated | `GLOBAL_API` |
| SCL-027 | `item_status` enum = `NORMAL` / `UNLIST` / `BANNED` / `DELETED` (+ `REVIEWING?`) and its transitions | `STILL_UNVERIFIED` | SPD-011,SPD-010 | CORRECTED / PRIMARY_VERIFIED — `item_status` ∈ `NORMAL, BANNED, UNLIST, SELLER_DELETE, SHOPEE_DELETE, REVIEWING` (no plain `DELETED`); `add_item` accepts `UNLIST`|`NORMAL` | `GLOBAL_API` |
| SCL-028 | Whether editing a live item re-triggers review; whether a draft can be created via API | `UNVERIFIED` | (corpus) | PRIMARY_NOT_FOUND — full transition graph / whether an edit re-triggers `REVIEWING` not stated; draft-via-API = `UNLIST` + `scheduled_publish_time` only | `GLOBAL_API` |

## Variation / model contract (P0)

| Claim ID | Claim | Previous state | Evidence | New state | Scope |
|---|---|---|---|---|---|
| SCL-030 | Models are created by a **separate call** (`init_tier_variation` / `add_model`) after `add_item`; `add_model` returns `model_id`s | `SEARCH_INDEXED` · MEDIUM · MULTI_SOURCE | SPD-015,SPD-017 | PRIMARY_VERIFIED — models via `init_tier_variation` (structure + initial models, 'at most 50') then `add_model`; both take `item_id`, return `model_id`s; wait ≥5 s after item creation | `GLOBAL_API` |
| SCL-031 | Model is addressed by `item_id` + `model_id`; per-model `price`/`stock` via `*_list` entries carrying `model_id` | `SEARCH_INDEXED` · MEDIUM | SPD-016,SPD-027,SPD-028,SPD-020 | PRIMARY_VERIFIED — model addressed by `item_id`+`model_id`; per-model price/stock via `price_list`/`stock_list` entries carrying `model_id`; `model_id=0` = no-model item | `GLOBAL_API` |
| SCL-032 | Tier structure: ≤ 2 tiers, positional options, `tier_index[]`, tier-1 option images | `SEARCH_INDEXED`; cap `UNVERIFIED` | SPD-015,SPD-029 | CORRECTED / PRIMARY_VERIFIED — `standardise_tier_variation` (old `tier_variation` deprecated 2025-09-12); **max 2 tiers**; `tier_index[]`; option/name length from `get_item_limit` (sample 20/14); tier-1 option image supported | `GLOBAL_API` |
| SCL-033 | **Whether a no-variation Item has a hidden/default Model** | `UNRESOLVED` | SPD-011,SPD-016,SPD-027 | **PRIMARY_PARTIAL** — the `model_id = 0` "no model item" **addressing convention** is PRIMARY_VERIFIED for the documented operations (`update_price` / `update_stock` / `get_model_list`). Whether a fully queryable default-model *entity* exists for every no-variation item across every API is inferred from one FBS-context phrase (`is_fulfillment_by_shopee` — 'only has a default model') — not fully established | `GLOBAL_API` |
| SCL-034 | Max models per item; max options per tier; model SKU length | `UNVERIFIED` | SPD-015 | PRIMARY_PARTIAL / PRIMARY_CONFLICT — model list per call ≤50; total models per item `< 20 (50 for TW)` (error) vs request text 'at most 50'; options per tier ≤20; combinations ≤50; `model_sku` ≤100 chars | `GLOBAL_API` (likely `RESOURCE_DYNAMIC`) |
| SCL-035 | Post-sale mutability of tier structure | `UNVERIFIED` | SPD-015 | PRIMARY_VERIFIED — structure change supported (no↔1↔2 tiers) but LOCKED while under promotion (`error_cannt_*_in_promotion`); CNSC shops blocked; FBS item/model not editable | `GLOBAL_API` |

## Category contract (P0)

| Claim ID | Claim | Previous state | Evidence | New state | Scope |
|---|---|---|---|---|---|
| SCL-040 | Category tree via `get_category(language)`; prediction via `category_recommend(item_name)` | `SEARCH_INDEXED` · MEDIUM · MULTI_SOURCE | SPD-021,SPD-022 | PRIMARY_VERIFIED — `get_category` (param `language`, whole tree) → `category_id`, `parent_category_id`, names, `has_children`; `category_recommend` (param `item_name` + optional `product_cover_image`) → ranked `category_id[]` + `ds_cat_rcmd_id` | `GLOBAL_API` |
| SCL-041 | Listing must sit under a **leaf** category | `UNRESOLVED` (likely, not proven) | SPD-021,SPD-010 | PRIMARY_PARTIAL — leaf = `has_children:false`; no explicit 'must list under a leaf' sentence, but implied by per-leaf brand/attribute + `error_invalid_category` 'L1 and L2 do not match' | `GLOBAL_API` |
| SCL-042 | Category exposes a status / eligibility / seller-restriction field | `UNVERIFIED` (no `listing_allowed` analogue — do not import ML's) | SPD-021,SPD-010 | PRIMARY_VERIFIED — no `listing_allowed`-style field; restriction surfaces at `add_item` as `error_category_is_block` 'Category is restricted' / `error_forbidden_category` / 'Category is blocked for CB seller' | `GLOBAL_API` |
| SCL-043 | Category-change behaviour on a live listing | `UNVERIFIED` | (corpus) | PRIMARY_NOT_FOUND — category-change-on-live-listing behaviour not documented in corpus | `GLOBAL_API` |

## Attribute contract (P0)

| Claim ID | Claim | Previous state | Evidence | New state | Scope |
|---|---|---|---|---|---|
| SCL-050 | Attributes for a category via `get_attribute_tree(category_id, language)` | `SEARCH_INDEXED` · MEDIUM (name) | SPD-023 | PRIMARY_VERIFIED — `GET /api/v2/product/get_attribute_tree`, `category_id_list` (max 20) + `language` (BR pt-BR/en) → per-category `attribute_tree[]` | `GLOBAL_API` |
| SCL-051 | Recommended set via `get_recommend_attribute(category_id, item_name)` | `SEARCH_INDEXED` | SPD-024 | PRIMARY_VERIFIED — `get_recommend_attribute` is a distinct endpoint for recommended (non-mandatory) attributes → feeds QUALITY | `GLOBAL_API` |
| SCL-052 | `is_mandatory` is a real field expressing per-category requiredness | `STILL_UNVERIFIED` | SPD-023 | CORRECTED / PRIMARY_VERIFIED — field is **`mandatory`** (bool) in `get_attribute_tree` (Phase 02.1/02.2 `is_mandatory` was `add_item` prose / `get_brand_list`); static per-category | `GLOBAL_API` |
| SCL-053 | Attribute schema: `attribute_id`, name, `input_type` set, `value_id` list, `value_unit`, multi-select, free-text | `STILL_UNVERIFIED` | SPD-023 | PRIMARY_VERIFIED — `attribute_id`, `mandatory`, `name`, `attribute_value_list[{value_id,name,value_unit,child_attribute_list}]`, `attribute_info{input_type(1-5), input_validation_type(0-4), format_type(1-2), date_format_type(0-1), attribute_unit_list, max_value_count, is_oem, support_search_value}` | `GLOBAL_API` |
| SCL-054 | Whether requiredness can depend on other field values or on the shop | `UNVERIFIED` (no mechanism seen) | SPD-023 | PRIMARY_VERIFIED — no conditional-requiredness mechanism; `mandatory` is a static per-category boolean. `support_search_value` → `v2.product.search_attribute_value_list` (not in corpus) | `GLOBAL_API` |
| SCL-055 | Regulatory attributes (INMETRO/ANVISA numbers) validated vs merely stored; role of `get_cert_rule` | `UNVERIFIED` | SPD-010,SPD-011 | PRIMARY_PARTIAL — regulated inputs are ordinary attributes (NCC/BSMI/FDA errors in `add_item`); PH `certification_info` via `v2.product.get_product_certification_rule` (page not in corpus); INMETRO/ANVISA not seen | `BRAZIL` |

## Brand contract (P0)

| Claim ID | Claim | Previous state | Evidence | New state | Scope |
|---|---|---|---|---|---|
| SCL-060 | Brands via `get_brand_list(category_id, offset, page_size, status)` — per category | `SEARCH_INDEXED` · MEDIUM | SPD-025 | PRIMARY_VERIFIED — `get_brand_list` per **leaf category**; params `offset`,`page_size`(max 100),`category_id`,`status`(1 normal/2 pending),`language`; returns `brand_list[{brand_id,original_brand_name,display_brand_name}]`,`has_next_page`,`next_offset` | `GLOBAL_API` |
| SCL-061 | Brand requiredness is **category-dependent** (→ CONDITIONAL, not universal CORE) | `SEARCH_INDEXED` (inferred from per-category `get_brand_list`) | SPD-025,SPD-010 | PRIMARY_VERIFIED — `get_brand_list` returns `is_mandatory` (bool) for the brand attribute per category; `add_item` ALWAYS requires `brand.brand_id`+`brand.original_brand_name` (True/True) | `BRAZIL` |
| SCL-062 | "Sem marca" / No-Brand is a supported value (first in the list) | `SEARCH_INDEXED` | SPD-010,SPD-025 | PRIMARY_VERIFIED — 'No Brand' = `brand_id: 0` + `original_brand_name` 'No Brand' (`error_invalid_brand` 'Brand ID value should be "0"') | `BRAZIL` |
| SCL-063 | `register_brand` is an API resource; approval / rejection / auto-revert behaviour | `SEARCH_INDEXED` (name observed) / `UNVERIFIED` (behaviour) | SPD-026 | PRIMARY_VERIFIED — `POST /api/v2/product/register_brand` (`original_brand_name`≤254, `category_list` L1/L2 max 50, `brand_region`, images); QC-reviewed (`error_busi_pending_qc`); blacklist rules; `unsupport_region_for_register_brand` for some markets | `GLOBAL_API` |
| SCL-064 | Brand-authorisation-gated (IP) categories | `UNVERIFIED` | SPD-010,SPD-026 | PRIMARY_PARTIAL — `add_item.authorised_brand_id` field exists (authorised reseller brand); `error_brand_forbidden` at add_item; explicit IP-authorisation category list not in corpus | `BRAZIL` |

## Identifier contract (P0)

| Claim ID | Claim | Previous state | Evidence | New state | Scope |
|---|---|---|---|---|---|
| SCL-070 | GTIN/EAN/UPC/ISBN API field, item-vs-model scope, per-category requiredness, format validation, duplicate rules | `UNVERIFIED` | SPD-010,SPD-015,SPD-017,SPD-020,SPD-029,SPD-011 | PRIMARY_VERIFIED — `gtin_code` string on add_item/add_model/init_tier_variation; **model-level** ('only TW seller and BR local seller'); item-level for default-model items; GS1 8-14 digits | `GLOBAL_API` / `BRAZIL` |
| SCL-071 | Whether a structured "legitimately no barcode" mechanism exists (an `EMPTY_GTIN_REASON` analogue) | `UNVERIFIED` (none seen — do not invent) | SPD-010,SPD-029 | PRIMARY_VERIFIED — 'Item without GTIN' ⇒ `gtin_code = "00"`; validation via `get_item_limit.gtin_limit.gtin_validation_rule` ∈ Mandatory / Flexible / Optional. No `EMPTY_GTIN_REASON` analogue — the mechanism IS the literal '00' | `GLOBAL_API` |

## Price contract (P0)

| Claim ID | Claim | Previous state | Evidence | New state | Scope |
|---|---|---|---|---|---|
| SCL-080 | `update_price(item_id, price_list)` — item-level or per-model; batch; BRL | `SEARCH_INDEXED` · MEDIUM | SPD-027 | PRIMARY_VERIFIED — `POST /api/v2/product/update_price`; `item_id`+`price_list[]` (1-50) {`model_id`(0=no model),`original_price` float}; item-level blocked when item has models | `GLOBAL_API` |
| SCL-081 | Price bounds (`price_limit`) and their resolution source | `UNVERIFIED` (source unconfirmed) | SPD-029 | CORRECTED / PRIMARY_VERIFIED — bounds from `get_item_limit.price_limit{min_limit,max_limit}` (sample 5.5 / 1e7) — SHOP+CATEGORY DYNAMIC; `wholesale_price_threshold_percentage` also returned | `RESOURCE_DYNAMIC` / `CATEGORY_DYNAMIC` |
| SCL-082 | `original_price` vs promo/`current_price`; price-range display when models differ | `SEARCH_INDEXED` | SPD-027,SPD-011,SPD-020 | PRIMARY_VERIFIED — `original_price` vs `current_price` (= promo price); `price_info` not returned on item when it has models (use `get_model_list`); BR: 2 decimal places allowed | `GLOBAL_API` |

## Stock contract (P0)

| Claim ID | Claim | Previous state | Evidence | New state | Scope |
|---|---|---|---|---|---|
| SCL-090 | `update_stock(item_id, stock_list)` — item-level or per-model; batch | `SEARCH_INDEXED` · MEDIUM | SPD-028 | PRIMARY_VERIFIED — `POST /api/v2/product/update_stock`; one `item_id`/call; `stock_list[]` (1-50) {`model_id`(0=no model),`seller_stock[]`{`location_id` opt,`stock`}}; updates only `seller_stock`; `failure_list`/`success_list` per model | `GLOBAL_API` |
| SCL-091 | Absolute-set vs incremental/delta write | `UNVERIFIED` (assumed absolute) | SPD-028 | PRIMARY_VERIFIED — write is **absolute** ('new stock info'); `normal_stock` deprecated/offlined 2022-10-31 ('use seller_stock') | `GLOBAL_API` |
| SCL-092 | `seller_stock` / `shop_stock` split; available vs reserved fields | `UNVERIFIED` | SPD-011,SPD-020 | PRIMARY_VERIFIED — reads return `stock_info_v2`: `summary_info{total_reserved_stock,total_available_stock}`, `seller_stock[{location_id,stock,if_saleable}]`, `shopee_stock[{location_id,stock}]`, `advance_stock{sellable,in_transit}` (PH/VN/ID/MY only) | `GLOBAL_API` |
| SCL-093 | Whether BR stock has a warehouse / `location_id` dimension (multi-warehouse) | `UNRESOLVED` (none observed; absence ≠ proof) | SPD-028,SPD-015 | CORRECTED / PRIMARY_VERIFIED — multi-warehouse IS supported: `location_id` from `v2.shop.get_warehouse_detail` (not in corpus); optional when shop has no warehouse; cannot mix stock structures; WMS shops blocked (`error_wms_shop_block_upate_stock`); FBS/B2C: normal stock must = 0 | `BRAZIL` |
| SCL-094 | Concurrency / optimistic-lock / version token on stock writes | `UNVERIFIED` (last-write-wins assumed) | SPD-028 | PRIMARY_PARTIAL — no optimistic-lock token; batch returns per-model `failure_list`/`success_list`; reserved-stock guard ('Total stock must be more than reserved stock') | `GLOBAL_API` |

## Logistics contract (P0)

| Claim ID | Claim | Previous state | Evidence | New state | Scope |
|---|---|---|---|---|---|
| SCL-100 | `logistics/get_channel_list`, `logistics/get_address` | `SEARCH_INDEXED` | (corpus) | STILL_UNVERIFIED — `v2.logistics.get_channel_list` / `get_address(_list)` pages NOT in corpus; `add_item.logistic_info` fields known: `logistic_id`(True),`enabled`(True),`is_free`,`size_id`(when fee_type=SIZE_SELECTION),`shipping_fee`(when fee_type=CUSTOM_PRICE) | `GLOBAL_API` |
| SCL-101 | Days-to-ship limit resource lives in the **`logistics`** service (not `product`); exact name | `UNVERIFIED` (Phase 02.2 corrected the filing) | SPD-029,SPD-010 | CORRECTED / PRIMARY_PARTIAL — DTS limits ARE in `get_item_limit.days_to_ship_limit{min,max,non_pre_order_days_to_ship}` (category-scoped); `add_item` prose also names `get_dts_limit` (page not in corpus); `error_category_dts` / `error_param_dts_exceeds_max_limit` | `GLOBAL_API` |
| SCL-102 | `weight` / `dimension` requiredness (channel- vs category-driven); pre-order window; dangerous-goods flag | `UNVERIFIED` | SPD-029,SPD-010 | PRIMARY_VERIFIED — `weight` (KG) required on add_item; `dimension` cm required when provided; `get_item_limit.weight_limit.weight_mandatory` / `dimension_limit.dimension_mandatory` (category); `pre_order{is_pre_order,days_to_ship}`; `item_dangerous` (ID/MY only); pre-order category-gated (`error_param_category_not_support_pre_order`) | `GLOBAL_API` / `BRAZIL` |

## Media contract (P1)

| Claim ID | Claim | Previous state | Evidence | New state | Scope |
|---|---|---|---|---|---|
| SCL-110 | `media_space/upload_image` → `image_id`; `media_space/upload_video` → `video_upload_id`; persist ids not URLs | `SEARCH_INDEXED` | SPD-010,SPD-022 | PRIMARY_PARTIAL — `add_item.image.image_id_list` (required); image ids from `v2.media_space.upload_image` (page not in corpus); `video_upload_id` (one) from video upload API; persist ids | `GLOBAL_API` |
| SCL-111 | Image count / dimensions / ratio / byte cap; item image field; cover; tier-option image mapping | `UNVERIFIED` (1–9 / 1:1 / 3:4 / ≈350² are seller-education only) | SPD-029,SPD-010 | PRIMARY_PARTIAL — count from `get_item_limit.item_image_count_limit` (sample 1-9, DYNAMIC); `image_ratio` '1:1' default / '3:4' whitelist-only; `promotion_images` one, ratio must be 3:4. Pixel/byte/type/cover rules NOT in corpus (media_space page) | `RESOURCE_DYNAMIC` / `MARKET_STATIC` |
| SCL-112 | Listing-video duration / aspect / size / moderation (distinct from Shopee Video / Live) | `UNVERIFIED` | SPD-010,SPD-011 | PRIMARY_PARTIAL — one `video_upload_id` per item; read returns `video_info{video_url,thumbnail_url,duration}`. Duration/aspect/size constraints NOT in corpus | `GLOBAL_API` |

## Limits & validator (P0/P1)

| Claim ID | Claim | Previous state | Evidence | New state | Scope |
|---|---|---|---|---|---|
| SCL-120 | `get_item_limit` is a **shop / item listing quota (listing capacity)**, not the source of title/description/image/variation/price limits | `SEARCH_INDEXED` (Phase 02.2 correction; not primary-confirmed) | SPD-029 | CORRECTED / PRIMARY_VERIFIED — `get_item_limit` IS the dynamic source of field limits (name/desc/image/price/stock/tier/DTS + weight/dimension/size-chart/GTIN requiredness). `item_count_limit{max}` (sample 5000) is the shop listing quota — one field among many | `SHOP_DYNAMIC` (hypothesised) |
| SCL-121 | The real resolution source for title / description / image / variation / price limits | `UNVERIFIED` | SPD-029 | CORRECTED / PRIMARY_VERIFIED — resolution source for title/description/image/variation/price/DTS limits = `v2.product.get_item_limit` (optional `category_id`) | `UNKNOWN_SCOPE` |
| SCL-122 | Title max/min length | `SEARCH_INDEXED` (≈255/256 — provisional, unlocked) | SPD-029,SPD-010 | CORRECTED — source is `get_item_limit.item_name_length_limit` (DYNAMIC per shop+category); doc **sample** min 5 / max 100 (a sample, not a constant). The prior ≈255/256 fixed guess has **no primary basis and is superseded** by the dynamic source — not "disproven", just unfounded. Errors `error_title_exceeds_max_length` / `error_item_name_is_too_short` / `error_title_character_forbidden` | `UNKNOWN_SCOPE` |
| SCL-123 | Description max length; `extended` description availability | `SEARCH_INDEXED` (≈5,000 — provisional, unlocked) | SPD-029 | CORRECTED — source is `get_item_limit.item_description_length_limit` (DYNAMIC); doc **sample** min 10 / max 2000 (a sample, not a constant). The prior ≈5,000 fixed guess has **no primary basis and is superseded**. `extended_description_limit` separate object (text len, image num/width/height/aspect); hashtags ≤ 18; extended description = whitelist sellers | `UNKNOWN_SCOPE` |
| SCL-124 | A dedicated pre-publication validator / dry-run / precheck exists | `NO_DEDICATED_VALIDATOR_FOUND` (large SDK surface searched; absence not proven) | (corpus) | PRIMARY_NOT_FOUND_IN_PRIMARY_CORPUS — no validate/precheck/dry-run/content-diagnosis/violation endpoint in the 35 PDFs; the `add_item`/`update_item` response + enumerated business error codes are the gate. Not asserting absence. | `GLOBAL_API` |

## Moderation / diagnosis (P1)

| Claim ID | Claim | Previous state | Evidence | New state | Scope |
|---|---|---|---|---|---|
| SCL-130 | `get_item_violation_info(item_id)` — per-item policy violations (post-publication enforcement) | `SEARCH_INDEXED` (name observed) | (corpus) | PRIMARY_NOT_FOUND — `get_item_violation_info` not in corpus. `get_item_base_info` exposes `item_status` (`BANNED`) + `deboost` (search-ranking lowered) | `GLOBAL_API` |
| SCL-131 | `get_item_content_diagnosis_result` / `get_item_list_by_content_diagnosis` — post-publication **QUALITY** diagnostic, **not** a pre-publication gate | `SEARCH_INDEXED` (name observed) | (corpus) | PRIMARY_NOT_FOUND — content-diagnosis endpoints not in corpus (Phase 02.2 SEARCH_INDEXED names remain unverified) | `GLOBAL_API` |
| SCL-132 | BR penalty-point / account-health API availability | `UNVERIFIED` | SPD-028,SPD-027 | PRIMARY_PARTIAL — `error_seller_under_penalty` 'The shop is under penalty' blocks price/stock edits; no penalty-points API page in corpus | `BRAZIL` |

## Mapping / structure (design — depends on the above)

| Claim ID | Claim | Previous state | Evidence | New state | Scope |
|---|---|---|---|---|---|
| SCL-140 | `internal product_id ↔ Shopee item_id`; `internal variant_id / SKU ↔ Shopee model_id` | design candidate | SPD-010, SPD-015, SPD-020, SPD-011 | PRIMARY_VERIFIED (design) — `item_id` from `add_item`; `model_id` from `init_tier_variation`/`add_model`; persist Shopee ids; SKU stored in `item_sku` / `model_sku` | — |
| SCL-141 | No-variation mapping: `variant_id → item_id`, OR `variant_id → hidden model_id` if one exists | `UNRESOLVED` (depends on SCL-033) | SPD-016, SPD-027, SPD-011 | CORRECTED / PRIMARY_VERIFIED (mapping) — `variant_id → (item_id, model_id = 0)`; the `model_id = 0` addressing convention is verified; no separate hidden id is exposed. (Entity-existence nuance: SCL-033 is `PRIMARY_PARTIAL`.) | — |
| SCL-142 | `v2.product.*` (local, in scope) vs `v2.global_product.*` (cross-border, out of scope) — no leakage | Phase 02.2A scope guard | SPD-010 … SPD-029 | PRIMARY_VERIFIED — corpus is entirely `v2.product.*`; `v2.global_product.*` exists for CB (out of scope); `ssp_id` (Shopee Standard Product) noted as a catalogue-like concept | — |

## Promotion rule

Move a row's `New state` to `PRIMARY_VERIFIED` **only** when a named `SPD-xxx`
artifact's text directly supports that specific claim, at a stated scope. A page
proving an endpoint path does not thereby verify its numeric limits, its Brazil
eligibility, or its stock semantics — those are separate rows.
