# Claim Registry — Phase 02.3

last_updated: 2026-08-30
status: **All claims awaiting primary evidence. No claim is `PRIMARY_VERIFIED`.**

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
| SCL-001 | Auth is OAuth per shop: `partner_id`+`partner_key` → redirect → `code` → `access_token` + `refresh_token` | `SEARCH_INDEXED` · MEDIUM · MULTI_SOURCE | (none) | awaiting primary | `GLOBAL_API` |
| SCL-002 | `access_token` lifetime ≈ 4 h; `refresh_token` ≈ 30 d, rotates | `SEARCH_INDEXED` | (none) | awaiting primary | `GLOBAL_API` |
| SCL-003 | Request signing = HMAC-SHA256 over `partner_id + api_path + timestamp + [access_token] + [shop_id]`; timestamp in seconds | `SEARCH_INDEXED`; **exact base string `UNVERIFIED`** | (none) | awaiting primary | `GLOBAL_API` |
| SCL-004 | Version prefix is `/api/v2`; token endpoints `auth/token/get`, `auth/access_token/get` | `SEARCH_INDEXED` · MEDIUM | (none) | awaiting primary | `GLOBAL_API` |
| SCL-005 | Hosts: global `partner.shopeemobile.com`, sandbox `partner.test-stable.shopeemobile.com` | `SEARCH_INDEXED` | (none) | awaiting primary | `GLOBAL_API` |
| SCL-006 | `openplatform.shopee.com.br` is the canonical BR Product API base URL | `SEARCH_INDEXED` · LOW · SINGLE_SOURCE (existence signal only) | (none) | awaiting primary | `BRAZIL` |

## Brazil eligibility (P0)

| Claim ID | Claim | Previous state | Evidence | New state | Scope |
|---|---|---|---|---|---|
| SCL-010 | Shopee BR offers an Open Platform application/onboarding process for sellers/integrators | `SEARCH_INDEXED` (article titles 3445, 27314 — bodies unread) | (none) | awaiting primary | `BRAZIL` |
| SCL-011 | Which seller/account types are eligible for BR Product API access | `UNRESOLVED` | (none) | awaiting primary | `BRAZIL` |
| SCL-012 | Whether BR Product API access requires approved-partner status / manual approval | `UNRESOLVED` (evidence leans approval-gated; **not** `CONFIRMED_RESTRICTED`) | (none) | awaiting primary | `BRAZIL` |
| SCL-013 | Whether a BR sandbox/test environment is available | `UNRESOLVED` | (none) | awaiting primary | `BRAZIL` |
| SCL-014 | Which API function-groups / permissions a BR partner app is granted | `UNRESOLVED` | (none) | awaiting primary | `BRAZIL` |

## Item contract (P0)

| Claim ID | Claim | Previous state | Evidence | New state | Scope |
|---|---|---|---|---|---|
| SCL-020 | `add_item` creates a listing and returns `item_id` | `SEARCH_INDEXED` · MEDIUM · MULTI_SOURCE | (none) | awaiting primary | `GLOBAL_API` |
| SCL-021 | `add_item` minimal required fields = `item_name`, `description`, `category_id`, `price`, `stock` | `SEARCH_INDEXED` (single SDK doc) | (none) | awaiting primary | `GLOBAL_API` |
| SCL-022 | Full `add_item` required/conditional field set (images, weight, dimension, logistics, brand, attribute_list, condition, item_sku, …) | `UNVERIFIED` | (none) | awaiting primary | `GLOBAL_API` |
| SCL-023 | `add_item` / `update_item` response shape: `item_id`, warnings, failure list, `request_id`, field-level errors | `UNVERIFIED` | (none) | awaiting primary | `GLOBAL_API` |
| SCL-024 | `get_item_base_info` takes `item_id_list`, max 50 per call | `SEARCH_INDEXED` · SINGLE_SOURCE / provisional | (none) | awaiting primary | `GLOBAL_API` |
| SCL-025 | `product_id` (seen on one integrator page) is a local alias for `item_id` — or a `v2.global_product` / cross-border concept | `UNRESOLVED` | (none) | awaiting primary | `UNKNOWN_MARKET_SCOPE` |
| SCL-026 | `item_sku` is seller-set, mutable, buyer-invisible | `UNVERIFIED` | (none) | awaiting primary | `GLOBAL_API` |
| SCL-027 | `item_status` enum = `NORMAL` / `UNLIST` / `BANNED` / `DELETED` (+ `REVIEWING?`) and its transitions | `STILL_UNVERIFIED` | (none) | awaiting primary | `GLOBAL_API` |
| SCL-028 | Whether editing a live item re-triggers review; whether a draft can be created via API | `UNVERIFIED` | (none) | awaiting primary | `GLOBAL_API` |

## Variation / model contract (P0)

| Claim ID | Claim | Previous state | Evidence | New state | Scope |
|---|---|---|---|---|---|
| SCL-030 | Models are created by a **separate call** (`init_tier_variation` / `add_model`) after `add_item`; `add_model` returns `model_id`s | `SEARCH_INDEXED` · MEDIUM · MULTI_SOURCE | (none) | awaiting primary | `GLOBAL_API` |
| SCL-031 | Model is addressed by `item_id` + `model_id`; per-model `price`/`stock` via `*_list` entries carrying `model_id` | `SEARCH_INDEXED` · MEDIUM | (none) | awaiting primary | `GLOBAL_API` |
| SCL-032 | Tier structure: ≤ 2 tiers, positional options, `tier_index[]`, tier-1 option images | `SEARCH_INDEXED`; cap `UNVERIFIED` | (none) | awaiting primary | `GLOBAL_API` |
| SCL-033 | **Whether a no-variation Item has a hidden/default Model** | `UNRESOLVED` | (none) | awaiting primary | `GLOBAL_API` |
| SCL-034 | Max models per item; max options per tier; model SKU length | `UNVERIFIED` | (none) | awaiting primary | `GLOBAL_API` (likely `RESOURCE_DYNAMIC`) |
| SCL-035 | Post-sale mutability of tier structure | `UNVERIFIED` | (none) | awaiting primary | `GLOBAL_API` |

## Category contract (P0)

| Claim ID | Claim | Previous state | Evidence | New state | Scope |
|---|---|---|---|---|---|
| SCL-040 | Category tree via `get_category(language)`; prediction via `category_recommend(item_name)` | `SEARCH_INDEXED` · MEDIUM · MULTI_SOURCE | (none) | awaiting primary | `GLOBAL_API` |
| SCL-041 | Listing must sit under a **leaf** category | `UNRESOLVED` (likely, not proven) | (none) | awaiting primary | `GLOBAL_API` |
| SCL-042 | Category exposes a status / eligibility / seller-restriction field | `UNVERIFIED` (no `listing_allowed` analogue — do not import ML's) | (none) | awaiting primary | `GLOBAL_API` |
| SCL-043 | Category-change behaviour on a live listing | `UNVERIFIED` | (none) | awaiting primary | `GLOBAL_API` |

## Attribute contract (P0)

| Claim ID | Claim | Previous state | Evidence | New state | Scope |
|---|---|---|---|---|---|
| SCL-050 | Attributes for a category via `get_attribute_tree(category_id, language)` | `SEARCH_INDEXED` · MEDIUM (name) | (none) | awaiting primary | `GLOBAL_API` |
| SCL-051 | Recommended set via `get_recommend_attribute(category_id, item_name)` | `SEARCH_INDEXED` | (none) | awaiting primary | `GLOBAL_API` |
| SCL-052 | `is_mandatory` is a real field expressing per-category requiredness | `STILL_UNVERIFIED` | (none) | awaiting primary | `GLOBAL_API` |
| SCL-053 | Attribute schema: `attribute_id`, name, `input_type` set, `value_id` list, `value_unit`, multi-select, free-text | `STILL_UNVERIFIED` | (none) | awaiting primary | `GLOBAL_API` |
| SCL-054 | Whether requiredness can depend on other field values or on the shop | `UNVERIFIED` (no mechanism seen) | (none) | awaiting primary | `GLOBAL_API` |
| SCL-055 | Regulatory attributes (INMETRO/ANVISA numbers) validated vs merely stored; role of `get_cert_rule` | `UNVERIFIED` | (none) | awaiting primary | `BRAZIL` |

## Brand contract (P0)

| Claim ID | Claim | Previous state | Evidence | New state | Scope |
|---|---|---|---|---|---|
| SCL-060 | Brands via `get_brand_list(category_id, offset, page_size, status)` — per category | `SEARCH_INDEXED` · MEDIUM | (none) | awaiting primary | `GLOBAL_API` |
| SCL-061 | Brand requiredness is **category-dependent** (→ CONDITIONAL, not universal CORE) | `SEARCH_INDEXED` (inferred from per-category `get_brand_list`) | (none) | awaiting primary | `BRAZIL` |
| SCL-062 | "Sem marca" / No-Brand is a supported value (first in the list) | `SEARCH_INDEXED` | (none) | awaiting primary | `BRAZIL` |
| SCL-063 | `register_brand` is an API resource; approval / rejection / auto-revert behaviour | `SEARCH_INDEXED` (name observed) / `UNVERIFIED` (behaviour) | (none) | awaiting primary | `GLOBAL_API` |
| SCL-064 | Brand-authorisation-gated (IP) categories | `UNVERIFIED` | (none) | awaiting primary | `BRAZIL` |

## Identifier contract (P0)

| Claim ID | Claim | Previous state | Evidence | New state | Scope |
|---|---|---|---|---|---|
| SCL-070 | GTIN/EAN/UPC/ISBN API field, item-vs-model scope, per-category requiredness, format validation, duplicate rules | `UNVERIFIED` | (none) | awaiting primary | `GLOBAL_API` / `BRAZIL` |
| SCL-071 | Whether a structured "legitimately no barcode" mechanism exists (an `EMPTY_GTIN_REASON` analogue) | `UNVERIFIED` (none seen — do not invent) | (none) | awaiting primary | `GLOBAL_API` |

## Price contract (P0)

| Claim ID | Claim | Previous state | Evidence | New state | Scope |
|---|---|---|---|---|---|
| SCL-080 | `update_price(item_id, price_list)` — item-level or per-model; batch; BRL | `SEARCH_INDEXED` · MEDIUM | (none) | awaiting primary | `GLOBAL_API` |
| SCL-081 | Price bounds (`price_limit`) and their resolution source | `UNVERIFIED` (source unconfirmed) | (none) | awaiting primary | `RESOURCE_DYNAMIC` / `CATEGORY_DYNAMIC` |
| SCL-082 | `original_price` vs promo/`current_price`; price-range display when models differ | `SEARCH_INDEXED` | (none) | awaiting primary | `GLOBAL_API` |

## Stock contract (P0)

| Claim ID | Claim | Previous state | Evidence | New state | Scope |
|---|---|---|---|---|---|
| SCL-090 | `update_stock(item_id, stock_list)` — item-level or per-model; batch | `SEARCH_INDEXED` · MEDIUM | (none) | awaiting primary | `GLOBAL_API` |
| SCL-091 | Absolute-set vs incremental/delta write | `UNVERIFIED` (assumed absolute) | (none) | awaiting primary | `GLOBAL_API` |
| SCL-092 | `seller_stock` / `shop_stock` split; available vs reserved fields | `UNVERIFIED` | (none) | awaiting primary | `GLOBAL_API` |
| SCL-093 | Whether BR stock has a warehouse / `location_id` dimension (multi-warehouse) | `UNRESOLVED` (none observed; absence ≠ proof) | (none) | awaiting primary | `BRAZIL` |
| SCL-094 | Concurrency / optimistic-lock / version token on stock writes | `UNVERIFIED` (last-write-wins assumed) | (none) | awaiting primary | `GLOBAL_API` |

## Logistics contract (P0)

| Claim ID | Claim | Previous state | Evidence | New state | Scope |
|---|---|---|---|---|---|
| SCL-100 | `logistics/get_channel_list`, `logistics/get_address` | `SEARCH_INDEXED` | (none) | awaiting primary | `GLOBAL_API` |
| SCL-101 | Days-to-ship limit resource lives in the **`logistics`** service (not `product`); exact name | `UNVERIFIED` (Phase 02.2 corrected the filing) | (none) | awaiting primary | `GLOBAL_API` |
| SCL-102 | `weight` / `dimension` requiredness (channel- vs category-driven); pre-order window; dangerous-goods flag | `UNVERIFIED` | (none) | awaiting primary | `GLOBAL_API` / `BRAZIL` |

## Media contract (P1)

| Claim ID | Claim | Previous state | Evidence | New state | Scope |
|---|---|---|---|---|---|
| SCL-110 | `media_space/upload_image` → `image_id`; `media_space/upload_video` → `video_upload_id`; persist ids not URLs | `SEARCH_INDEXED` | (none) | awaiting primary | `GLOBAL_API` |
| SCL-111 | Image count / dimensions / ratio / byte cap; item image field; cover; tier-option image mapping | `UNVERIFIED` (1–9 / 1:1 / 3:4 / ≈350² are seller-education only) | (none) | awaiting primary | `RESOURCE_DYNAMIC` / `MARKET_STATIC` |
| SCL-112 | Listing-video duration / aspect / size / moderation (distinct from Shopee Video / Live) | `UNVERIFIED` | (none) | awaiting primary | `GLOBAL_API` |

## Limits & validator (P0/P1)

| Claim ID | Claim | Previous state | Evidence | New state | Scope |
|---|---|---|---|---|---|
| SCL-120 | `get_item_limit` is a **shop / item listing quota (listing capacity)**, not the source of title/description/image/variation/price limits | `SEARCH_INDEXED` (Phase 02.2 correction; not primary-confirmed) | (none) | awaiting primary | `SHOP_DYNAMIC` (hypothesised) |
| SCL-121 | The real resolution source for title / description / image / variation / price limits | `UNVERIFIED` | (none) | awaiting primary | `UNKNOWN_SCOPE` |
| SCL-122 | Title max/min length | `SEARCH_INDEXED` (≈255/256 — provisional, unlocked) | (none) | awaiting primary | `UNKNOWN_SCOPE` |
| SCL-123 | Description max length; `extended` description availability | `SEARCH_INDEXED` (≈5,000 — provisional, unlocked) | (none) | awaiting primary | `UNKNOWN_SCOPE` |
| SCL-124 | A dedicated pre-publication validator / dry-run / precheck exists | `NO_DEDICATED_VALIDATOR_FOUND` (large SDK surface searched; absence not proven) | (none) | awaiting primary | `GLOBAL_API` |

## Moderation / diagnosis (P1)

| Claim ID | Claim | Previous state | Evidence | New state | Scope |
|---|---|---|---|---|---|
| SCL-130 | `get_item_violation_info(item_id)` — per-item policy violations (post-publication enforcement) | `SEARCH_INDEXED` (name observed) | (none) | awaiting primary | `GLOBAL_API` |
| SCL-131 | `get_item_content_diagnosis_result` / `get_item_list_by_content_diagnosis` — post-publication **QUALITY** diagnostic, **not** a pre-publication gate | `SEARCH_INDEXED` (name observed) | (none) | awaiting primary | `GLOBAL_API` |
| SCL-132 | BR penalty-point / account-health API availability | `UNVERIFIED` | (none) | awaiting primary | `BRAZIL` |

## Mapping / structure (design — depends on the above)

| Claim ID | Claim | Previous state | Evidence | New state | Scope |
|---|---|---|---|---|---|
| SCL-140 | `internal product_id ↔ Shopee item_id`; `internal variant_id / SKU ↔ Shopee model_id` | design candidate (depends on SCL-020, SCL-030, SCL-033) | (none) | not locked | — |
| SCL-141 | No-variation mapping: `variant_id → item_id`, OR `variant_id → hidden model_id` if one exists | `UNRESOLVED` (depends on SCL-033) | (none) | not locked | — |
| SCL-142 | `v2.product.*` (local, in scope) vs `v2.global_product.*` (cross-border, out of scope) — no leakage | Phase 02.2A scope guard | (none) | held | — |

## Promotion rule

Move a row's `New state` to `PRIMARY_VERIFIED` **only** when a named `SPD-xxx`
artifact's text directly supports that specific claim, at a stated scope. A page
proving an endpoint path does not thereby verify its numeric limits, its Brazil
eligibility, or its stock semantics — those are separate rows.
