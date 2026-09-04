# API & auth — Open Platform

last_reviewed: 2026-08-28
phase_02_2_reviewed: 2026-08-30
phase_02_3_reviewed: 2026-09-03
volatile: true
scope_note: >-
  **Phase 02.3 read 35 official Shopee Open Platform PDFs**
  (`docs/marketplaces/shopee/open-platform/`, registry
  `research/shopee-primary-docs/`). The v2 `product` + `auth` contract below is
  now largely `PRIMARY_VERIFIED` — see §0 for the primary corrections that
  supersede the older `SEARCH_INDEXED` body, and `claim-registry.md` /
  `extraction-report.md` for per-fact SPD citations. Still **not** covered by a
  primary page: the `logistics` service (`get_channel_list`, `get_address`,
  `get_warehouse_detail`), `media_space` upload pages, `search_attribute_value_list`,
  `get_recommend_attribute` full schema, `update_item` full field list. Do not
  build a client / auth flow / publish pipeline from this file — Phase 02.3 is
  evidence, not implementation.

## 0. Phase 02.3 primary corrections (authoritative — supersede the body below)

Source: `research/shopee-primary-docs/` (SPD-001 … SPD-029). All `PRIMARY_VERIFIED`
unless noted.

### Hosts (every API-reference page, "Request Address")
| Environment | Region | Base |
|---|---|---|
| Production | Global | `https://partner.shopeemobile.com/api/v2/<path>` |
| Production | **Brazil** | `https://openplatform.shopee.com.br/api/v2/<path>` |
| Production | China Mainland | `https://openplatform.shopee.cn/api/v2/<path>` |
| Sandbox | Global | `https://openplatform.sandbox.test-stable.shopee.sg/api/v2/<path>` |
| Auth (link) | Global / **BR** / CN | `https://open.shopee.com/auth` · `https://open.shopee.com.br/auth` · `https://open.shopee.cn/auth` (+ `/cancel_auth`; sandbox under `open.sandbox.test-stable.shopee.*`) |

(The older `partner.test-stable.shopeemobile.com` sandbox host is **CORRECTED** to
`openplatform.sandbox.test-stable.shopee.sg`.)

### Auth contract (SPD-001)
- Flow: build authorization link → seller authorizes shop(s) → `POST /api/v2/auth/token/get`
  (`code` → `access_token` + `refresh_token`) → refresh via `POST /api/v2/auth/access_token/get`.
- Link params: `partner_id`, `auth_type` (`seller` | `supplier` | `user`),
  `redirect_uri` (domain must match the Console Redirect URL Domain),
  `response_type=code`, `state` (optional CSRF).
- **Sign base string** (path without host, in order): Shop APIs =
  `partner_id + api_path + timestamp + access_token + shop_id`; Merchant APIs =
  `… + merchant_id`; Public APIs (`auth_partner`, `token/get`,
  `access_token/get`) = `partner_id + api_path + timestamp`. HMAC-SHA256, key =
  **partner key**, hex lowercase. `partner_id` + `timestamp` + `sign` in the
  query; other params in the JSON body.
- **Lifetimes**: `access_token` **4 h**; `refresh_token` **30 d** (single-use per
  `shop_id`/`merchant_id`); auth `code` **10 min** single-use; sign `timestamp`
  **5 min**; old `access_token` valid **+5 min** after a refresh; authorization
  validity **≤ 360 days** (SPD-006 onboarding guide says 365 — minor
  `PRIMARY_CONFLICT`).
- Account types: Shop account (1 shop) / Main account (many) / Sub-account
  (cannot authorize). Redirect returns `code` + `shop_id` **or** `code` +
  `main_account_id`.
- **App-category = permission set**, chosen at app creation, immutable; each API
  page lists its allowed "APP types". Calling outside the category → Permission
  Denied.

### `get_item_limit` — the dynamic limit source (SPD-029) — CORRECTS Phase 02.2
`GET /api/v2/product/get_item_limit`, optional `category_id`. Its `response`
object returns (values are per shop+category, **DYNAMIC** — the numbers are the
doc's samples, not constants): `item_name_length_limit` (title),
`item_description_length_limit`, `extended_description_limit`,
`item_image_count_limit` (sample 1–9), `price_limit`, `stock_limit`,
`wholesale_price_threshold_percentage`, `tier_variation_name_length_limit`,
`tier_variation_option_length_limit`, `days_to_ship_limit`,
`weight_limit.weight_mandatory`, `dimension_limit.dimension_mandatory`,
`size_chart_limit.size_chart_mandatory`, `gtin_limit.gtin_validation_rule`
(`Mandatory` / `Flexible` / `Optional`), **and** `item_count_limit` (the shop's
total-listing quota — one field, not the whole purpose). **Phase 02.2's "probably
just a quota, not the limit source" is rejected.**

### Entity / lifecycle (SPD-010, SPD-011, SPD-015)
- `add_item` (`POST`) → returns `item_id` (int64). `init_tier_variation` /
  `add_model` take `item_id`, return `model_id`s.
- **`item_status` enum**: `NORMAL`, `BANNED`, `UNLIST`, `SELLER_DELETE`,
  `SHOPEE_DELETE`, `REVIEWING` (CORRECTS the `DELETED` guess).
- **Max variation tiers = 2** (`error_param` "The level of tier-variation over
  2"). Old `tier_variation` structure **deprecated 2025-09-12** → use
  `standardise_tier_variation`.
- **`model_id = 0` = "no model item"**; Shopee keeps an internal **default
  model** for no-variation items.
- `get_item_base_info.item_id_list` limit **[0, 50]**; `price_list` /
  `stock_list` write length **1–50**; model list per `init_tier_variation` /
  `add_model` call ≤ **50**; total models per item **< 20 (50 for TW)** —
  `PRIMARY_CONFLICT` with "at most 50"; `model_sku` ≤ **100** chars.

### Attribute / brand / GTIN (SPD-023, SPD-025, SPD-026)
- `get_attribute_tree` (`GET`): `category_id_list` (**max 20**) + `language` (BR:
  `pt-BR`/`en`). Field is **`mandatory`** (bool), not `is_mandatory`.
  `input_type` = `SINGLE_DROP_DOWN`1 / `SINGLE_COMBO_BOX`2 / `FREE_TEXT_FILED`3 /
  `MULTI_DROP_DOWN`4 / `MULTI_COMBO_BOX`5. No conditional-requiredness mechanism.
- `get_brand_list` (`GET`): per **leaf category**; `status` 1 normal / 2 pending;
  `page_size` max 100; returns `is_mandatory` for the brand attribute.
  `add_item` **always** requires `brand.brand_id` + `brand.original_brand_name`;
  **"No Brand" ⇒ `brand_id: 0`**.
- `register_brand` (`POST`): real; QC-reviewed (`error_busi_pending_qc`);
  `unsupport_region_for_register_brand` for some markets (BR support
  `PRIMARY_NOT_FOUND`).
- **GTIN**: `gtin_code` string; **`"00"` = "Item without GTIN"**; validation from
  `get_item_limit.gtin_limit.gtin_validation_rule`; model-level (BR + TW).
  No `EMPTY_GTIN_REASON` analogue — the mechanism is the literal `"00"`.

## 1. Status (Phase 02.3)

> **The `v2.product.*` + `auth` listing contract HAS been verified** against 35
> official Shopee Open Platform pages (§0; `research/shopee-primary-docs/`).

- §0 carries the `PRIMARY_VERIFIED` facts (hosts, auth + signing contract, token
  lifetimes, `get_item_limit`, entity / lifecycle, attribute / brand / GTIN).
  Where §0 and any older row below disagree, **§0 wins.**
- Still **not** covered by a primary page — treat as `SEARCH_INDEXED` at best,
  `UNVERIFIED` for detail: the `logistics` service (`get_channel_list`,
  `get_address`, `get_warehouse_detail`), `media_space` upload pages,
  `search_attribute_value_list`, `get_recommend_attribute` full schema,
  `update_item` full field list, penalty / content-diagnosis / violation APIs.
- Community SDK / integrator endpoint names still do **not** upgrade a rule.
- The Skill output remains **publish-agnostic**: a listing draft + audit JSON.
  Building a client / auth flow / publish pipeline is Phase 02.4+, not this file.

## 2. Auth model — `PRIMARY_VERIFIED` (see §0 for the authoritative contract)

The exact auth + signing contract is in §0 (SPD-001). Summary:

| Element | Value | Verification |
|---|---|---|
| Partner identity | `partner_id` + `partner_key` | `PRIMARY_VERIFIED` (§0) |
| Shop authorisation | build auth link → seller authorises shop(s) → `code` → `POST /api/v2/auth/token/get`; one app authorises many shops | `PRIMARY_VERIFIED` (§0) |
| Access token | `access_token`, lifetime **4 h** | `PRIMARY_VERIFIED` (§0) |
| Refresh token | `refresh_token`, lifetime **30 d**, single-use per `shop_id` / `merchant_id` | `PRIMARY_VERIFIED` (§0) |
| Auth `code` / sign `timestamp` | `code` **10 min** single-use; timestamp window **5 min**; old access token valid **+5 min** after a refresh | `PRIMARY_VERIFIED` (§0) |
| Authorization validity | **≤ 360 days** (SPD-006 onboarding guide says 365 — unresolved `PRIMARY_CONFLICT`; do not pick a side) | `PRIMARY_VERIFIED` with conflict |
| Token scoping | per `shop_id` (or `main_account_id`); stored per shop | `PRIMARY_VERIFIED` (§0) |
| Version prefix | `/api/v2` | `PRIMARY_VERIFIED` (§0) |
| Hosts | Global `partner.shopeemobile.com`; **Brazil `openplatform.shopee.com.br`**; CN `openplatform.shopee.cn`; Sandbox Global `openplatform.sandbox.test-stable.shopee.sg` (all `/api/v2/<path>`). Auth-link host for BR: `open.shopee.com.br/auth`. | `PRIMARY_VERIFIED` (§0) |
| Token exchange / refresh | `POST /api/v2/auth/token/get` · `POST /api/v2/auth/access_token/get` | `PRIMARY_VERIFIED` (§0) |
| Request signing | HMAC-SHA256, key = partner key, hex lowercase, over the ordered base string **per API type** — Shop: `partner_id + api_path + timestamp + access_token + shop_id`; Merchant: `… + merchant_id`; Public: `partner_id + api_path + timestamp`. `partner_id` + `timestamp` + `sign` in the query; timestamp in **seconds**. | `PRIMARY_VERIFIED` (§0) |
| Multi-shop grouping | `merchant_id` for Merchant APIs / SIP; `main_account_id` returned at redirect for a main account | `PRIMARY_VERIFIED` (§0) |
| App-category permissions | app category chosen at creation, immutable; each API page lists its allowed APP types; calling outside the category → Permission Denied | `PRIMARY_VERIFIED` (§0) |
| Local vs global catalogue | `v2.product.*` = a shop's own catalogue (BR listings); `v2.global_product.*` = cross-border seller catalogue — **out of scope** | `PRIMARY_VERIFIED` (SPD-010…029 corpus is entirely `v2.product.*`) |
| Rate limits | exist; values `PRIMARY_NOT_FOUND`; back off on a rate-limit error | `UNVERIFIED` |

## 3. Endpoint registry — v2 `product` service

> **Phase 02.3**: the `product` methods and their key params below are now
> **`PRIMARY_VERIFIED`** against the official reference pages (SPD-010 … SPD-029;
> see §0 for corrections). `media_space`, `logistics`, `br` and enforcement rows
> remain non-primary (pages not supplied). Where §0 and a row disagree, **§0
> wins.**

Paths / method names: `SEARCH_INDEXED` · MEDIUM confidence · MULTI_SOURCE (all
non-primary) — a **corroborated API contract candidate**, safe for provisional
mapping design, **not** locked and **not** primary-`CONFIRMED`. **All schemas
`UNVERIFIED`.** Paths are `/api/v2/product/<method>` unless noted. "Doc locator" =
`open.shopee.com/documents/v2/v2.product.<method>?module=89&type=1` (product
module = `89`) — the page to verify; **not read**. Evidence keys: S12
`QuoVadis86/shopee-sdk` (Go), S13 `congminh1254/shopee-sdk` (TS, method→locator
map + JSON schemas), S14 `rollout.com`, S15 `publicapis.io`/`api2cart.com`.
"Purpose" / params are what those sources say — do not code against these.

### Auth
| Method | Purpose | Key params | Evidence |
|---|---|---|---|
| `auth/token/get` | exchange `code` → `access_token` + `refresh_token` | `partner_id`, `code`, `shop_id`/`main_account_id` | S14, S15 |
| `auth/access_token/get` | refresh the access token (rotates refresh token) | `partner_id`, `refresh_token`, `shop_id` | S14, S15 |
| `shop/auth_partner` (redirect) | seller authorises the app for a shop → auth `code` | `partner_id`, `sign`, redirect URL | S15 |

### Category / attribute / brand (all `DYNAMIC` sources — never hardcode their outputs)
| Method | Purpose | Key params | Evidence |
|---|---|---|---|
| `category_recommend` | predict candidate categories from the item name | `item_name` | S12, S13 |
| `get_category` | per-region category tree (`category_id`, `parent_category_id`, names, `has_children`) | `language` | S12, S13 |
| `get_attribute_tree` | attributes for a category — **corrects Phase 02.1 `get_attributes`** | `category_id`, `language` | S13 (S12: `get_attribute_tree` w/ `category_id_list`) |
| `get_recommend_attribute` | suggested (quality) attributes for a category + name | `category_id`, `item_name` | S12, S13 |
| `search_attr_value` | search attribute values | — | S12 |
| `get_brand_list` | brands for a **leaf** category (`brand_id`, names; No-Brand = `brand_id: 0`); returns `is_mandatory` for the brand attribute | `category_id`, `offset`, `page_size` (max 100), `status` (1/2), `language` | SPD-025 |
| `register_brand` | submit a seller/manufacturer brand — **Phase 01 said "not confirmed"; a name now exists** | — | S12 |
| `get_item_limit` | **the dynamic limit source** (SPD-029, `PRIMARY_VERIFIED` — see §0). `GET`, optional `category_id`; `response` returns `item_name_length_limit`, `item_description_length_limit`, `extended_description_limit`, `item_image_count_limit`, `price_limit`, `stock_limit`, `tier_variation_name_length_limit`, `tier_variation_option_length_limit`, `days_to_ship_limit`, `weight_limit`, `dimension_limit`, `size_chart_limit`, `gtin_limit.gtin_validation_rule`, **and** `item_count_limit` (the shop's total-listing quota — one field, not the whole purpose). | optional `category_id` | SPD-029 |
| `get_weight_rec` | weight recommendation | — | S12 |
| `get_size_chart_list` / `get_size_chart_detail` | size-chart support (fashion) | — | S12 |
| `get_cert_rule` | per-category certification rules (regulatory) — verify for `compliance.md` | `category_id`? | S12 |

### Item lifecycle
| Method | Purpose | Key params | Evidence |
|---|---|---|---|
| `add_item` | **create a listing** → returns `item_id` | `item_name`, `description`, `category_id`, `price`, `stock` (+ more, `UNVERIFIED`) | S12, S13 |
| `update_item` | edit listing fields | `item_id` + changed fields | S12, S13, S14 |
| `delete_item` | delete a listing | `item_id` | S13 |
| `unlist_item` | list / unlist toggle | `item_id`, `unlist` (bool) | S12, S13 |
| `get_item_base_info` | core listing fields | `item_id_list` limit **[0, 50]** (SPD-011, `PRIMARY_VERIFIED`). No `product_id` request parameter exists in the primary corpus — the integrator-page `product_id` is `PRIMARY_NOT_FOUND` as a local param; the read also returns `ssp_id` (Shopee Standard Product, catalogue-like — gap G6) and `deboost` | SPD-011 |
| `get_item_extra_info` | sales / views / likes | `item_id_list` | S12, S13 |
| `get_item_list` | listing ids by `item_status` | `offset`, `page_size`, `item_status` | S13 |
| `search_item` | search shop listings | `offset`, `page_size` | S12, S13 |
| `get_item_violation_info` | per-item policy violations — **`NOT PRIMARY VERIFIED — DO NOT RELY ON FOR EXECUTION`**; a Phase 02.2 `SEARCH_INDEXED` community-SDK name, **not** in the 35-PDF corpus. Use `get_item_base_info.item_status` (`BANNED`) + `deboost` instead. | `item_id` | S12, S13 (non-primary) |
| `get_item_content_diagnosis_result` | per-item content diagnosis — **`NOT PRIMARY VERIFIED — DO NOT RELY ON FOR EXECUTION`**; Phase 02.2 `SEARCH_INDEXED` name only, not in the corpus. | `item_id` | S12, S13 (non-primary) |
| `get_item_list_by_content_diagnosis` | items filtered by diagnosis status — **`NOT PRIMARY VERIFIED`**, as above. | `diagnosis_status`, `offset`, `page_size` | S12, S13 (non-primary) |

*Phase 02.1 listed `update_item_sku` — retained as `UNVERIFIED` (name from S7 only; not seen in S12/S13).*

### Variation / model
| Method | Purpose | Key params | Evidence |
|---|---|---|---|
| `init_tier_variation` | set the tier structure + initial models (one call) | `item_id`, `tier_variation`, `model` | S12, S13 |
| `update_tier_variation` | edit tier option names / images | `item_id`, `tier_variation` | S12, S13 |
| `add_model` | add models (combinations) → returns `model_id`s | `item_id`, `model_list` | S12, S13 |
| `update_model` | edit a model | `item_id`, `model` | S12, S13 |
| `delete_model` | remove a model | `item_id`, `model_id` | S12, S13 |
| `get_model_list` | list an item's models + tier structure | `item_id` | S12, S13 |
| `get_variations` | read variations | `item_id` | S12 |

Lifecycle (corroborated API contract candidate — non-primary): `add_item` first →
then `init_tier_variation` **or** `add_model` to create models (a **separate
call** from item creation). Usable for provisional mapping design; not an
implementation contract until primary docs are read.

### Price / stock
| Method | Purpose | Key params | Evidence |
|---|---|---|---|
| `update_price` (+ batch) | price — item-level, or per model via `price_list` entries carrying `model_id` | `item_id`, `price_list` | S12, S13 |
| `update_stock` (+ batch) | stock — item-level, or per model via `stock_list` entries carrying `model_id`; absolute-set **assumed** | `item_id`, `stock_list` | S12, S13 |

### Media (`media_space` service)
| Method | Purpose | Evidence |
|---|---|---|
| `media_space/upload_image` | upload image → `image_id` (CDN URLs expire — persist the id) | S12, S15 |
| `media_space/upload_video` / `get_video_upload_result` | listing video → `video_upload_id`; constraints `UNVERIFIED` | S12 |

### Logistics (`logistics` service — names beyond the first two `SEARCH_INDEXED` at best)
| Method | Purpose | Evidence |
|---|---|---|
| `logistics/get_channel_list` | logistics channels enabled for the shop | S15 |
| `logistics/get_address` | pickup / return / default `address_id` | S15 |
| days-to-ship limit | **`get_item_limit.days_to_ship_limit`** (`{min, max, non_pre_order_days_to_ship}`, category-scoped) is the `PRIMARY_VERIFIED` source (SPD-029). `add_item` prose also names a dedicated `get_dts_limit` page — **not in the corpus**. The `logistics`-service filing from Phase 02.2 is superseded for the *limit* values. | SPD-029 |

### Brazil-specific (`br` service — `SEARCH_INDEXED`, S12; BR-only)
| Method | Purpose | Evidence |
|---|---|---|
| `br/query_shop_enrollment_status` | whether a shop is enrolled in BR operations | S12 |
| `br/query_shop_invoice_error` | nota-fiscal / invoice errors | S12 |
| `br/query_shop_block_status` · `br/query_sku_block_status` | BR shop / SKU block status (enforcement) | S12 |

### Enforcement / validation
| Resource | Status |
|---|---|
| `public/get_shop_penalty` / account-health | `PRIMARY_NOT_FOUND` for BR (no penalty-points API page in the corpus). `error_seller_under_penalty` on `update_price` / `update_stock` + `get_item_base_info.deboost` are the primary-verified signals. |
| pre-publication validate / dry-run | `NO_DEDICATED_VALIDATOR_FOUND_IN_PRIMARY_CORPUS` — no `validate` / `dry-run` / `precheck` / content-diagnosis / violation endpoint in the 35 PDFs. **Not asserting absence.** The `add_item` / `update_item` response + the large enumerated business error-code set (`PRIMARY_VERIFIED`) is the gate. `get_item_content_diagnosis_result` / `get_item_violation_info` are `PRIMARY_NOT_FOUND` — do not rely on them. |

## 4. Brazil access (gap G2) — `PRIMARY_PARTIAL` (Phase 02.3)

Primary evidence (SPD-003, SPD-004, SPD-005, SPD-008, SPD-009, SPD-035):

- **BR Product API availability is a derived conclusion** (`PRIMARY_DERIVED`, not
  a single explicit Shopee declaration): every `v2.product.*` reference page
  lists the Brazil host `https://openplatform.shopee.com.br/api/v2/…`; the auth
  doc lists `https://open.shopee.com.br/auth`; `add_item` carries BR-specific
  fields (`tax_info` NCM/CFOP/CEST/CSOSN/PIS/COFINS/ICMS, `export_cfop`,
  model-level `gtin_code` "BR local seller", 2-decimal prices). Taken together
  these explicitly document a **BR production Product API** — but no page states
  "the Product API is available to Brazil sellers" as one sentence.
- **Two approval gates:**
  1. **Developer profile** — a login alone can't create apps; you apply for a
     developer profile and are **approved by Shopee's internal criteria**
     (`developer-guide/12`, criteria page not supplied — `PRIMARY_NOT_FOUND`).
     The BR Developer Guide lists two developer types for this journey:
     `Registered Business Seller` and `Third-party Partner (ISV)` (SPD-004).
     (SPD-035's BR SPI App guide separately names `Individual Seller` as a
     developer type in the Seller-Logistics / SPI context — a different scope;
     do not force the two lists into a conflict.)
  2. **Go-Live review** — Production access requires submitting a Product Brief +
     Redirect URL domains + IP whitelist + IT-assets declaration; **Shopee
     reviews before approving Production**. Sandbox is independent and available
     first.
- **App category = permission set**, chosen at creation, immutable. Common:
  Registered Business Seller → `Seller In-House System`; ISV → `ERP System`.
  Both list `add_item` / `update_stock` / `get_category` / `get_attribute_tree`
  / `get_brand_list` / `get_item_limit` etc. in "APP types that can call this
  API". **No separate per-category whitelist is documented in the primary corpus
  for these listing-API app categories** — unlike `Swarm ERP`, `Brand
  Membership`, `Auto Parts Installation` **SPI** apps, which SPD-035 explicitly
  marks "Whitelist Only". Absence of a documented whitelist for the listing
  categories is **not** proof that no whitelist exists — and the developer
  profile itself is still approval-gated (above).

**Consequence:** `resolve_open_platform_br_access` becomes an **account/app
readiness** check: does this shop's authorising app have (a) an approved BR
developer profile of an eligible type, (b) an app in a listing-capable category,
(c) a completed Go-Live for Production? Pending → `EXECUTION = REVIEW`; a
confirmed missing gate → `EXECUTION = FAIL` for the target op (fall back to
manual/Seller-Center publish). **Never** a global `FAIL` on "availability", and
**no universal claim** that every BR developer is eligible.

`PRIMARY_NOT_FOUND`: the exact developer-profile approval criteria; whether any
per-category whitelist applies to the listing APIs; whether `register_brand` is
supported in BR.

<!-- superseded Phase 02.2 wording follows for history -->

**(Phase 02.2, superseded):** who may register a partner app for BR; which API
function-groups a BR partner is granted; sandbox availability for BR; the
production-approval bar; account-type restrictions. The non-primary evidence
*leans* toward "available via an application / approval program" rather than
unavailable — **but the state is not recorded as `CONFIRMED_RESTRICTED`** until
primary eligibility / onboarding material is ingested (Phase 02.3).

Operational posture (until an approved BR developer account + Go-Live are in hand):

- Assume **manual / Seller-Center publishing**. The Skill produces a draft +
  audit for a human to enter, not an API payload to POST.
- `EXECUTION_STATUS` for any operation that needs the API stays `REVIEW`
  ("Open Platform access for this BR shop not confirmed") — never `FAIL` on that
  basis alone (Adversarial Test H).
- Do not design queues, token stores, retry logic or a publish pipeline.

## 5. Dynamic check registry

Every `DYNAMIC` decision the Skill makes is one of these checks. **No endpoint is
hardcoded** — each names a resource whose own `verification` state is in §3.
`pending` always → `REVIEW` (never `FAIL`); only an **executed** check that
**confirms a mandatory incompatibility** → `FAIL`.

**Phase 02.3 audit (brief §17) of each check — `PRIMARY_JUSTIFIED_DYNAMIC` /
`STILL_NEEDED_UNVERIFIED` / `STATIC_PRIMARY_RULE` / `QUALITY_ONLY` / `KEEP`:**

| Check | Why | Source (primary) | Pending | Confirmed-incompatibility | Verification / audit |
|---|---|---|---|---|---|
| `resolve_leaf_category` | listing must sit under a valid leaf | `get_category` (`has_children:false` = leaf) + `category_recommend` | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` (`error_invalid_category` / `error_category_is_block`) | `PRIMARY_VERIFIED` (SPD-021/022); leaf-only sentence `PRIMARY_PARTIAL` — **KEEP** |
| `resolve_category_attributes` | which attributes are `mandatory` for the category | **`get_attribute_tree`** (`category_id_list` ≤ 20; field **`mandatory`** bool) | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` (mandatory unmet — `error_less_required_attribute`) | `PRIMARY_VERIFIED` (SPD-023) — **PRIMARY_JUSTIFIED_DYNAMIC** |
| `resolve_recommended_attributes` | completeness beyond the minimum | `get_recommend_attribute` (SPD-024, skimmed) | `QUALITY = REVIEW` | — never `FAIL` | `PRIMARY_PARTIAL` — **QUALITY_ONLY** |
| `resolve_brand_requirement` | is the brand attribute required for this leaf; is the value resolvable | `get_brand_list` (per **leaf**; `status` 1/2; returns `is_mandatory`) + `add_item` (brand object always required) | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` (brand object missing → `error_less_required_brand`; `brand_id`≠0 unresolvable) | `PRIMARY_VERIFIED` (SPD-025/010) — **PRIMARY_JUSTIFIED_DYNAMIC** |
| `resolve_brand_authorisation` | IP-gated brand authorisation | `add_item.authorised_brand_id`; `error_brand_forbidden`; explicit category list `PRIMARY_NOT_FOUND` | `PUBLICATION = REVIEW` | `PUBLICATION`/`EXECUTION = FAIL` (`error_brand_forbidden`) | `STILL_NEEDED_UNVERIFIED` (SPD-010, SPD-026) |
| `resolve_identifier_requirement` | GTIN requiredness / mode | **`get_item_limit.gtin_limit.gtin_validation_rule`** (`Mandatory`/`Flexible`/`Optional`); `gtin_code="00"` = no GTIN; model-level, BR+TW | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` (`Mandatory` + absent, or `error_param_validate`) — never invent a code | `PRIMARY_VERIFIED` (SPD-029/010) — **PRIMARY_JUSTIFIED_DYNAMIC** |
| `resolve_title_limit` | `item_name` min/max | **`get_item_limit.item_name_length_limit`** (per shop+category) | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` (`error_title_exceeds_max_length` / `error_item_name_is_too_short`) | `PRIMARY_VERIFIED` source (SPD-029) — **PRIMARY_JUSTIFIED_DYNAMIC** (CORRECTED: source was "UNVERIFIED") |
| `resolve_description_limit` | description length; `extended` availability | **`get_item_limit.item_description_length_limit`** + `extended_description_limit`; `extended` = whitelist sellers; hashtags ≤ 18 | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` (`error_desc_length_min_limit` / over max / `error_desc_hash_tag_over_limit`) | `PRIMARY_VERIFIED` source (SPD-029) — **PRIMARY_JUSTIFIED_DYNAMIC** (CORRECTED) |
| `resolve_image_limits` | image count / ratio | **`get_item_limit.item_image_count_limit`** (count); `add_item.image_ratio` `"1:1"`/`"3:4"` (**3:4 = whitelist**); pixel/byte/type/cover rules → `media_space` page **not in corpus** | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` (`error_image_num_min` / over max) | count = `PRIMARY_JUSTIFIED_DYNAMIC` (SPD-029); other specs `STILL_NEEDED_UNVERIFIED` |
| `resolve_variation_limits` | tiers / options / models | **max 2 tiers = `STATIC_PRIMARY_RULE`**; option-count ≤ 20, combos ≤ 50 (errors); total models `< 20 (50 TW)` — `PRIMARY_CONFLICT`; name/option-name length via `get_item_limit` | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` (over 2 tiers; over resolved caps) | `PRIMARY_VERIFIED` (SPD-015/029) — **KEEP + note the 20/50 conflict for Phase 02.4** |
| `resolve_price_bounds` | min/max price | **`get_item_limit.price_limit`** (per shop+category) | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` (`error_price_exceed_min/max_limit`) | `PRIMARY_VERIFIED` source (SPD-029) — **PRIMARY_JUSTIFIED_DYNAMIC** (CORRECTED) |
| `resolve_dts_limit` | days-to-ship range | **`get_item_limit.days_to_ship_limit`** (category) — CORRECTED: it **is** in `get_item_limit`; `add_item` prose also names `get_dts_limit` (page not in corpus) | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` (`error_category_dts` / `error_param_dts_exceeds_max_limit`) | `PRIMARY_PARTIAL` (SPD-029/010) — **PRIMARY_JUSTIFIED_DYNAMIC** |
| `resolve_logistics` | ≥ 1 enabled channel; weight present | `add_item.logistic_info` (`logistic_id`,`enabled` required); `weight` required; `get_channel_list` page **not in corpus** | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` (`error_invalid_logistic_info` / `error_param` "Invalid Weight") | `STILL_NEEDED_UNVERIFIED` for the channel-list resource (SPD-010) — **KEEP** |
| `resolve_condition_allowed` | `condition = USED` allowed? | `add_item.condition` `NEW`/`USED` (field confirmed); per-category allow-list `PRIMARY_NOT_FOUND` | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` if a resolved rule disallows | `PRIMARY_PARTIAL` (SPD-010) — **STILL_NEEDED_UNVERIFIED** (the gate) |
| `resolve_inventory_model` | single pool vs multi-warehouse; write shape | `update_stock` writes **`seller_stock`** absolute; `location_id` from **`v2.shop.get_warehouse_detail`** (page not in corpus); can't mix structures; WMS shops blocked (`error_wms_shop_block_upate_stock`); FBS/B2C normal stock = 0 | `EXECUTION = REVIEW` | `EXECUTION = FAIL` (`error_busi` "has multi warehouse, please input location id"; wrong structure; WMS-blocked) | `PRIMARY_VERIFIED` structure (SPD-028/015); warehouse-detail resource `STILL_NEEDED_UNVERIFIED` — **KEEP** |
| `resolve_shop_api_capability` | shop auth + token + app-category permission for the target op; item not `BANNED` / `SELLER_DELETE` / `SHOPEE_DELETE` for an update | token store / `auth/*` (4h/30d lifetimes) / `get_item_base_info.item_status` / app-category list on the endpoint page | `EXECUTION = REVIEW` | `EXECUTION = FAIL` (no valid token; Permission Denied for the category; item banned/deleted; `error_seller_under_penalty`) | `PRIMARY_VERIFIED` (SPD-001/007/011) — **KEEP** |
| `resolve_model_exists` | a model-scoped op needs an existing `model_id` (never for CREATE); no-variation → `model_id = 0` | `get_model_list` (`item_id`) | `EXECUTION = REVIEW` | `EXECUTION = FAIL` (no model) | `PRIMARY_VERIFIED` (SPD-020) — **KEEP** |
| `resolve_compliance_applicability` | prohibited / restricted / regulated status | Shopee BR policy (`compliance.md`); `add_item` `error_category_is_block` / `error_forbidden_category` / NCC/BSMI/FDA attribute errors; PH cert via `get_product_certification_rule` (page not in corpus) | `PUBLICATION`/`EXECUTION = REVIEW` | `FAIL` per `compliance.md` | `PRIMARY_PARTIAL` (SPD-010) — **KEEP** |
| `resolve_contact_diversion_clean` | no contact / external-diversion strings in the assembled payload | payload scan (INTERNAL) | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` if still present at publish (`CONTENT` stays `PASS` if dropped) | `INTERNAL` — **KEEP** |
| `resolve_open_platform_br_access` | approved BR developer profile (Registered Business Seller / ISV) + listing-capable app category + completed Go-Live (Production) | Open Platform Console / onboarding (SPD-003/004/005/009) — **not a runtime endpoint**; an operational precondition | `EXECUTION = REVIEW` | `EXECUTION = FAIL` for the target op (fall back to manual publish); **never** a global `FAIL` on "availability" | `PRIMARY_PARTIAL` (§4 — onboarding flow documented; profile-approval criteria `PRIMARY_NOT_FOUND`) — **KEEP, redefined** |
| `resolve_content_diagnosis` *(optional, post-publication)* | Shopee's own listing-quality diagnosis for an existing item | `get_item_content_diagnosis_result` — **NOT in the primary corpus**; Phase 02.2 SEARCH_INDEXED name only. `get_item_base_info.deboost` (search-ranking lowered) is a primary-verified proxy. | `QUALITY = REVIEW` | — (never `FAIL`) | `PRIMARY_NOT_FOUND` — **KEEP as optional; use `deboost` in the meantime** |

The same four readiness dimensions absorb every check — **no fifth dimension is
needed.**

## Sources

- **PRIMARY (Phase 02.3)** — `docs/marketplaces/shopee/open-platform/` PDFs
  `SPD-001` (Authorization & Authentication), `SPD-003`/`SPD-004`/`SPD-005`/
  `SPD-008`/`SPD-009` (BR developer journey), `SPD-035` (BR SPI App guide),
  `SPD-010` (`add_item`), `SPD-011` (`get_item_base_info`), `SPD-015`
  (`init_tier_variation`), `SPD-017` (`add_model`), `SPD-020` (`get_model_list`),
  `SPD-021` (`get_category`), `SPD-022` (`category_recommend`), `SPD-023`
  (`get_attribute_tree`), `SPD-025` (`get_brand_list`), `SPD-026`
  (`register_brand`), `SPD-027` (`update_price`), `SPD-028` (`update_stock`),
  `SPD-029` (`get_item_limit`). Official Shopee Open Platform, captured
  2026-09-01/03, read 2026-09-03. Registry: `research/shopee-primary-docs/`.
  Verification = `LIVE` / `PRIMARY_VERIFIED` for the facts in §0 and the
  `PRIMARY_JUSTIFIED_DYNAMIC` rows in §5.
- **S12** `github.com/QuoVadis86/shopee-sdk` (Go, "380+ endpoints / 15 domains")
  — community SDK — consulted 2026-08-28 — `SEARCH_INDEXED` — v2 `product` method
  names; region hosts incl. `openplatform.shopee.com.br`; `br` module; sign base
  string; `media_space` / `logistics` service split. Not a canonical contract.
- **S13** `github.com/congminh1254/shopee-sdk` (TS, "100% coverage") —
  `docs/managers/product.md` + `schemas/v2.product.*.json` — community SDK —
  consulted 2026-08-28 — `SEARCH_INDEXED` (higher quality: each method → path +
  required params + an `open.shopee.com/documents/v2/v2.product.<m>?module=89&type=1`
  locator; `product` vs `global_product` split).
- **S14** `rollout.com` Shopee integration guide — external — consulted
  2026-08-28 — `SEARCH_INDEXED` — `/api/v2` base; `get_item_base_info` param
  `product_id`, response `item_id`; `auth/token/get`; token 4 h.
- **S15** `publicapis.io/shopee-api`, `api2cart.com` — external — consulted
  2026-08-28 — `SEARCH_INDEXED` — `/api/v2` base; sandbox host; auth model;
  HMAC-SHA256; "route to the correct country endpoint".
- **S16** Shopee BR Centro de Educação do Vendedor — arts. 3445 ("Shopee Open
  API Platform | Passo a Passo de Solicitação"), 27314 ("Open Platform Shopee:
  Guia Prático de Integração") — Centro de Educação — consulted 2026-08-28 —
  `SEARCH_INDEXED` (title + snippet; body is SPA).
- **S17** `geckoapi.com.br` (BR, 2026) — external — consulted 2026-08-28 —
  `SEARCH_INDEXED` — BR store ops via "Open Platform … Aplicação e autorização
  conforme o programa oficial".
- Prior community SDKs / integrators — `wjp-letgo/shopeego`, `teacat/shopeego`
  (v1), `raviMukti/shopee-api-client`, `mu-hanz/shoapi`, `developer.inlinex.com.sg`
  — external — consulted 2026-08-28 — `SEARCH_INDEXED`.
- Shopee Open Platform API portal — `https://open.shopee.com` /
  `https://developer.shopee.com` — Open Platform — consulted 2026-08-28 —
  `UNVERIFIED` (**tool-blocked + SPA**) — the source that must replace every row
  above.
- `public/get_shop_penalty` / account-health — other Shopee markets — consulted
  2026-08-28 — `UNVERIFIED` for BR.
- Full analysis: `research/shopee-api-contract/phase-02.2-report.md`
  (§7 API registry, §15 limits, §27 dynamic-check audit, §29 corrections).
