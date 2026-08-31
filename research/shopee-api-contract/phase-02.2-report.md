# Shopee Open Platform — Phase 02.2 Primary API Contract Verification

research_date: 2026-08-28 – 2026-08-30
branch: research/shopee-api-contract
market_scope: Shopee **Brasil** (region BR). Global-API facts labelled `GLOBAL_API`.
author: product-create team
inputs: Phase 02.1 Skill (`.claude/skills/shopee-listing-best-practices/`), Phase 01
  discovery report (`research/shopee-listing-skill/discovery-report.md`).

---

## 1. Executive Result

**FINAL DECISION: `API CONTRACT PARTIALLY VERIFIED — USER PRIMARY ARTIFACT REQUIRED`.**

- **No primary Shopee source was read `LIVE`.** `open.shopee.com`,
  `developer.shopee.com` and `web.archive.org` are **blocked at the fetch-tool
  level** in this environment (not a network timeout — an explicit refusal). The
  Open Platform portal is additionally a client-rendered SPA: fetched through a
  Google-Translate proxy (`open-shopee-com.translate.goog`) it still returns only
  the page title, no body. Shopee BR seller/help sites behave the same way.
- **No official Shopee-maintained SDK or repository exists.** GitHub has only
  community SDKs (`QuoVadis86/shopee-sdk` Go, `congminh1254/shopee-sdk` TS,
  `raviMukti/shopee-api-client` PHP, `mu-hanz/shoapi` PHP, …).
- **What improved:** triangulation across two independent, well-structured
  community SDKs (one of which maps every method to a specific
  `open.shopee.com/documents/v2/...` locator), plus integrator guides and two
  Shopee **Brasil** seller-education article titles, lets us:
  - raise ~25 v2 `product` endpoint **names + paths + key params** from
    "scattered SEARCH_INDEXED" to "**corroborated SEARCH_INDEXED, MEDIUM**";
  - **correct** several Phase 02.1 details (`get_attributes` →
    `get_attribute_tree`; `get_item_limit` is very likely a shop listing-**quota**
    resource, **not** the source of title/description/image size limits;
    `get_dts_limit` is not a `product` resource; a per-item **violation** API and
    a **content-diagnosis** API do exist as names);
  - move Brazil Open Platform **existence** from "no evidence" to
    "well-triangulated" (dedicated `openplatform.shopee.com.br` host, a dedicated
    `br` API module, two official BR seller-education articles about applying for
    and integrating with the Open Platform) — while **eligibility terms remain
    unresolved**.
- **What did not improve:** every request/response **schema**, every **numeric
  limit**, the **auth signing base string**, the **stock/warehouse** model, the
  **item-status enum**, and **Brazil eligibility/approval scope** remain
  `UNVERIFIED`. Nothing here is safe to *lock*.

Rule-locking (Phase 02.3 category/attribute/brand rules) should **not** start
until a maintainer supplies primary artifacts (§42 list) or MCP/sandbox access.

---

## 2. Brazil Open Platform Availability

**State: `UNRESOLVED`** — but the weight of indirect evidence now points to
"available to sellers / integrators **through an application + approval
program**", i.e. the eventual answer is more likely `CONFIRMED_RESTRICTED` than
`CONFIRMED_UNAVAILABLE`.

| Evidence | Tier | What it shows | What it does **not** show |
|---|---|---|---|
| `openplatform.shopee.com.br` listed as the **Brazil region host** in `QuoVadis86/shopee-sdk` region config (alongside global `partner.shopeemobile.com`, `openplatform.shopee.cn`) | E | a BR-specific Open Platform endpoint exists | that an arbitrary BR seller can obtain credentials |
| `br.go` module in the same SDK: `br.query_shop_enrollment_status`, `br.query_shop_invoice_error` (nota fiscal), `br.query_shop_block_status`, `br.query_sku_block_status` | E | the API has **Brazil-specific operations** (fiscal, enrollment, block-status) — BR is a first-class region, not an afterthought | the auth/eligibility path to reach them |
| Shopee BR **Centro de Educação do Vendedor** art. **3445** — "Shopee Open API Platform \| Passo a Passo de Solicitação" (application step-by-step) | B | Shopee BR **documents an Open Platform application process** for sellers | body unreadable (SPA); who is eligible, what is approved |
| Shopee BR **Centro de Educação do Vendedor** art. **27314** — "Open Platform Shopee: Guia Prático de Integração" (practical integration guide, main flows + support) | B | Shopee BR **officially supports ERP / integrator integration** via the Open Platform | body unreadable; scope of granted API functions |
| BR integrator framing: "operação de loja: Open Platform … Aplicação e autorização conforme o programa oficial" | D | integrators treat BR Open Platform access as real but **program-gated** | the program's bar |

**Consequence for the Skill:** keep `resolve_open_platform_br_access` as a
`DYNAMIC` / account-context check. Do **not** flip `EXECUTION_STATUS` to a global
`FAIL`; an unresolved BR-access check is `REVIEW` with a manual/Seller-Center
fallback. Adversarial Test H satisfied.

---

## 3. Source Quality

| # | Source | Tier | Verification | Supports |
|---|---|---|---|---|
| S1 | `open.shopee.com` / `developer.shopee.com` Open Platform docs | A | **UNREACHABLE** (tool-blocked + SPA; proxy yields title only) | nothing read |
| S12 | `github.com/QuoVadis86/shopee-sdk` (Go, "380+ endpoints / 15 domains") | E | `SEARCH_INDEXED` | endpoint **names**; region hosts incl. `openplatform.shopee.com.br`; `br` module; sign base string; `item_status` context |
| S13 | `github.com/congminh1254/shopee-sdk` (TS, "100% endpoint coverage"); `docs/managers/product.md`; `schemas/v2.product.*.json` | E | `SEARCH_INDEXED` (higher quality — each method carries an `open.shopee.com/documents/v2/v2.product.<m>?module=89&type=1` locator and a JSON schema file) | v2 `product` method list + paths + required params; `product` vs `global_product` split |
| S14 | `rollout.com` Shopee integration guide | D | `SEARCH_INDEXED` | `/api/v2` base; `/product/get_item_base_info` takes `product_id`; response carries `item_id`; token endpoint; token 4 h |
| S15 | `publicapis.io/shopee-api`, `api2cart.com` | D | `SEARCH_INDEXED` | `/api/v2` base; sandbox host `partner.test-stable.shopeemobile.com`; `partner_id`+`partner_key`+`shop_id`+`access_token`; HMAC-SHA256; "Open Platform" vs "Seller API" framing; "route to the correct country endpoint" |
| S16 | Shopee BR Centro de Educação do Vendedor arts. 3445, 27314 (titles + snippets only) | B | `SEARCH_INDEXED` (title/snippet; body is SPA) | BR Open Platform application + integration processes exist |
| S17 | `geckoapi.com.br` (BR scraping-API blog, 2026) | E | `SEARCH_INDEXED` | BR store operations use "Open Platform … Aplicação e autorização conforme o programa oficial" |

Still no `LIVE`. Still no Tier-A or Tier-C evidence.

---

## 4. Authorization Model

Consistent across S12–S15 (`SEARCH_INDEXED`, MEDIUM):

```
Partner / App  (partner_id + partner_key)          — the integrator's credentials
      │  OAuth shop-authorization redirect → auth `code`
      ▼
Shop authorization  (per shop_id)
      │  code → token exchange
      ▼
access_token (per shop_id, ~4 h) + refresh_token (~30 d, rotates)
```

| Field | Meaning (reconstructed) | Assigned by | Scope | Required for product API? | Verification |
|---|---|---|---|---|---|
| `partner_id` | the integrator app id | Shopee (dev console) | app | yes (in every signed request) | `SEARCH_INDEXED` |
| `partner_key` / `partner_secret` | HMAC signing secret | Shopee | app | yes (signing) | `SEARCH_INDEXED` |
| `shop_id` | a seller storefront | Shopee | region-scoped | yes (shop-scoped ops) | `SEARCH_INDEXED` |
| `merchant_id` | groups shops (SIP / CB) | Shopee | merchant | only for merchant-scoped ops | `SEARCH_INDEXED` |
| `main_account_id` | the Shopee account owning shops | Shopee | account | used at token exchange in some flows | `UNVERIFIED` |
| `code` | short-lived authorization code | Shopee (redirect) | one-time | yes (to mint tokens) | `SEARCH_INDEXED` |
| `access_token` | shop-scoped bearer, ~4 h | Shopee | shop | yes | `SEARCH_INDEXED` |
| `refresh_token` | ~30 d, rotates on use | Shopee | shop | yes (renewal) | `SEARCH_INDEXED` |
| `region` | BR / global / CN / sandbox host selector | — | — | picks the base URL | `SEARCH_INDEXED` (S12) |

One app authorises **many** shops (S12, S15). Do not infer field semantics from
names beyond this.

---

## 5. Authentication Contract

| Element | Reconstructed | Verification |
|---|---|---|
| Global host | `https://partner.shopeemobile.com` | `SEARCH_INDEXED` |
| Sandbox host | `https://partner.test-stable.shopeemobile.com` | `SEARCH_INDEXED` |
| **Brazil host** | `https://openplatform.shopee.com.br` | `SEARCH_INDEXED` (S12 only — **corroborate**) |
| CN host | `https://openplatform.shopee.cn` | `SEARCH_INDEXED` |
| Version prefix | `/api/v2` | `SEARCH_INDEXED`, MEDIUM (S13 schema names + S14 + S15) |
| Token exchange | `POST /api/v2/auth/token/get` | `SEARCH_INDEXED` |
| Token refresh | `POST /api/v2/auth/access_token/get` | `SEARCH_INDEXED` |
| Signing | HMAC-SHA256; base string `partner_id + api_path + timestamp + [access_token] + [shop_id]`; timestamp in **seconds** | `SEARCH_INDEXED`; **exact composition / ordering `UNVERIFIED`** |
| Rate limits | exist; values unknown | `UNVERIFIED` |

**Do not implement signing.** Base-string details are not primary-confirmed.

---

## 6. API Version

**v2 is the current product API.** Every community SDK and integrator guide from
2024–2026 targets `/api/v2`; `v1` is referenced only as legacy. `congminh1254`
names its schema files `v2.product.*` and `v2.global_product.*`. No evidence of a
v3, and no evidence Brazil differs on version. `SEARCH_INDEXED`, MEDIUM.

Note a **local vs global** split (S13): `v2.product.*` (a shop's own catalogue)
vs `v2.global_product.*` (cross-border / CB seller catalogue that fans out to
regional items). Brazil listings are expected to be **`v2.product.*`**; the
`global_product` family is out of scope unless a CB model is later confirmed.

---

## 7. API Registry (v2 `product` — corroborated SEARCH_INDEXED, MEDIUM; **all schemas UNVERIFIED**)

Paths follow `/api/v2/product/<method>`. "Doc locator" = the
`open.shopee.com/documents/v2/v2.product.<method>?module=89&type=1` URL S13 maps
to each method (module `89` = product). Not read — a place to verify.

| Operation | Method / path | Key params (S13) | IDs required | Verification |
|---|---|---|---|---|
| List shop items | `get_item_list` | `offset`, `page_size`, `item_status` | shop | `SEARCH_INDEXED` |
| Read items | `get_item_base_info` | `item_id_list` (≤ 50) — also seen as `product_id` (S14) | `item_id` | `SEARCH_INDEXED` |
| Read item metrics | `get_item_extra_info` | `item_id_list` | `item_id` | `SEARCH_INDEXED` |
| Search shop items | `search_item` | `offset`, `page_size` | shop | `SEARCH_INDEXED` |
| **Create item** | `add_item` | `item_name`, `description`, `category_id`, `price`, `stock` (+ more — unverified) | none (produces `item_id`) | `SEARCH_INDEXED` |
| Update item | `update_item` | `item_id` + changed fields | `item_id` | `SEARCH_INDEXED` |
| Delete item | `delete_item` | `item_id` | `item_id` | `SEARCH_INDEXED` |
| Unlist / relist | `unlist_item` | `item_id`, `unlist` (bool) | `item_id` | `SEARCH_INDEXED` |
| Category tree | `get_category` | `language` | shop | `SEARCH_INDEXED`, MEDIUM |
| Category prediction | `category_recommend` | `item_name` | shop | `SEARCH_INDEXED` |
| **Attributes for a category** | `get_attribute_tree` | `category_id`, `language` | — | `SEARCH_INDEXED` — **corrects Phase 02.1 `get_attributes`** |
| Recommended attributes | `get_recommend_attribute` | `category_id`, `item_name` | — | `SEARCH_INDEXED` |
| Search attribute values | `search_attr_value` (name from S12) | — | — | `SEARCH_INDEXED` |
| Brands for a category | `get_brand_list` | `category_id`, `offset`, `page_size`, `status` | — | `SEARCH_INDEXED`, MEDIUM |
| Register a brand | `register_brand` (S12) | — | — | `SEARCH_INDEXED` — **Phase 01 said "not confirmed"; a name now exists** |
| **Shop listing limit** | `get_item_limit` | **"None" (S13)** / `item_name`,`category_id` (S12) — **conflict** | shop | `SEARCH_INDEXED` — **scope disputed; see §13/§15** |
| Weight recommendation | `get_weight_rec` (S12) | — | — | `SEARCH_INDEXED` |
| Size chart list / detail | `get_size_chart_list` / `get_size_chart_detail` (S12) | — | — | `SEARCH_INDEXED` |
| Init variation structure | `init_tier_variation` | `item_id`, `tier_variation`, `model` | `item_id` | `SEARCH_INDEXED`, MEDIUM |
| Edit variation structure | `update_tier_variation` | `item_id`, `tier_variation` | `item_id` | `SEARCH_INDEXED` |
| Add models | `add_model` | `item_id`, `model_list` | `item_id` (produces `model_id`s) | `SEARCH_INDEXED` |
| Update model | `update_model` | `item_id`, `model` | `item_id`, `model_id` | `SEARCH_INDEXED` |
| Delete model | `delete_model` | `item_id`, `model_id` | `item_id`, `model_id` | `SEARCH_INDEXED` |
| List models | `get_model_list` | `item_id` | `item_id` | `SEARCH_INDEXED`, MEDIUM |
| Read variations | `get_variations` (S12) | — | `item_id` | `SEARCH_INDEXED` |
| Update price | `update_price` | `item_id`, `price_list` | `item_id` (+ `model_id` in list) | `SEARCH_INDEXED`, MEDIUM |
| Update stock | `update_stock` | `item_id`, `stock_list` | `item_id` (+ `model_id` in list) | `SEARCH_INDEXED`, MEDIUM |
| **Per-item violations** | `get_item_violation_info` | `item_id` | `item_id` | `SEARCH_INDEXED` — **new; Phase 01 said none** |
| **Content diagnosis (per item)** | `get_item_content_diagnosis_result` | `item_id` | `item_id` | `SEARCH_INDEXED` — **new** |
| Items by diagnosis status | `get_item_list_by_content_diagnosis` | `diagnosis_status`, `offset`, `page_size` | shop | `SEARCH_INDEXED` — **new** |
| Certification rules | `get_cert_rule` (S12) | — | `category_id`? | `SEARCH_INDEXED` — regulatory; unverified |
| Vehicle compatibility | `get_all_vehicle_list` / `get_vehicle_comp_list` (S12) | — | — | `SEARCH_INDEXED` — auto-parts fitment |
| Kit / bundle items | `get_kit_limit` / `add_kit_item` / `update_kit_item` / `get_kit_item_info` (S12) | — | — | `SEARCH_INDEXED` — **concept Phase 02.1 does not model** |

**Not in the `product` service** (Phase 02.1 mis-filed): `get_dts_limit`
(days-to-ship) — expected under a `logistics` service (`get_channel_list`,
`get_address`, DTS) — `UNVERIFIED`. Media upload is a separate `media_space`
service (`upload_image`, `upload_video`) — `SEARCH_INDEXED` (S12 `media_space.go`).

**No endpoint path has been invented.** Where an operation is expected but no
name was corroborated, it is marked `UNVERIFIED`.

---

## 8. Entity Contract

| Entity | Phase 02.1 assumption | Phase 02.2 result | Verification | PF mapping |
|---|---|---|---|---|
| Partner / App | `partner_id` + key; one app → many shops | **PARTIALLY_CONFIRMED** | `SEARCH_INDEXED` | app credential store |
| Shop | `shop_id`, Shopee-assigned, region BR | **PARTIALLY_CONFIRMED** | `SEARCH_INDEXED` | account/shop mapping |
| Merchant | `merchant_id` for SIP/CB | **UNRESOLVED** (exists; relevance to BR listing unknown) | `SEARCH_INDEXED` | — |
| Item | the listing; `item_id` | **PARTIALLY_CONFIRMED** — `item_id` is the persisted id; some read endpoints accept it as `product_id` / `item_id_list` | `SEARCH_INDEXED`, MEDIUM | internal `product_id` ↔ `item_id` |
| Tier Variation | ≤ 2 tiers, positional, no id | **PARTIALLY_CONFIRMED** — `init_tier_variation`(`tier_variation`,`model`) + `update_tier_variation`; **max-2 still `UNVERIFIED`** | `SEARCH_INDEXED` | internal variation axes |
| Variation Option | ordered option per tier; tier-1 may carry an image | **UNRESOLVED** — structure implied, not confirmed | `UNVERIFIED` | axis values (not identity) |
| Model | sellable variant; `model_id`; addressed as `item_id`+`model_id` | **PARTIALLY_CONFIRMED** — `add_model` produces `model_id`s; `delete_model` takes `model_id`; per-model `price`/`stock` via `*_list` | `SEARCH_INDEXED`, MEDIUM | internal `variant_id` / SKU ↔ `model_id` |
| Category | leaf `category_id`; per-region tree via `get_category(language)` | **PARTIALLY_CONFIRMED** — method name corroborated; **leaf-only still not proven** | `SEARCH_INDEXED`, MEDIUM | 1 Item → 1 leaf |
| Attribute | per-category; `get_attribute_tree` | **CORRECTED** (name) — `get_attributes` → `get_attribute_tree(category_id, language)`; **field names `UNVERIFIED`** | `SEARCH_INDEXED` | — |
| Brand | `brand_id`; per category; `get_brand_list`; `register_brand` exists | **PARTIALLY_CONFIRMED** | `SEARCH_INDEXED` | — |
| Stock / Location | one seller pool per model (BR) | **UNRESOLVED** — `update_stock(stock_list)` corroborated; location dimension not seen but absence ≠ proof | `UNVERIFIED` | — |

---

## 9. Item Contract

- **Listing identity = `item_id`** (Shopee-assigned on `add_item`). Persist it.
  Read endpoints accept `item_id_list` (≤ 50) or, per some integrators,
  `product_id`. Treat `item`/`product` as the same entity; the **stored** id is
  `item_id`. (CONFIRMED as far as SEARCH_INDEXED allows.)
- **Create (`add_item`) minimal fields:** `item_name`, `description`,
  `category_id`, `price`, `stock`. The full required set (images, `weight`,
  `dimension`, `logistic_info`, `brand`, `attribute_list`, `condition`,
  `item_sku`, `seller_stock`, …) is **`UNVERIFIED`** — the Phase 02.1
  `CreateItem` concept list stands as a *concept*, not a schema.
- **`item_sku`** — seller-set, mutable, still believed buyer-invisible.
  `UNVERIFIED`.
- **Item-level vs model-level price/stock** — for a no-variation item, `update_price`/
  `update_stock` operate at item level; when models exist they carry `model_id`
  in the `price_list`/`stock_list`. `SEARCH_INDEXED`, MEDIUM.

---

## 10. Model / Variation Contract

- **Lifecycle:** `add_item` first → then either `init_tier_variation`
  (`tier_variation` + initial `model` list, one call) **or** `add_model`
  (`model_list`) to create models; models get `model_id`s in the response.
  **Models are generally a separate call from item creation.** (SEARCH_INDEXED,
  MEDIUM — satisfies Adversarial Test F / §35.)
- **Model addressing:** `update_model` needs `item_id` + `model`;
  `delete_model` needs `item_id` + `model_id`; `get_model_list` needs `item_id`.
- **`tier_index` / positional options / images-per-tier-1-option** — believed as
  Phase 01, **not** re-confirmed. `UNVERIFIED`.
- **No-variation item (§18 / Test E):** whether Shopee maintains an internal
  default model is **UNRESOLVED** — no endpoint or field seen that exposes one;
  `get_model_list` on a no-variation item's behaviour is unverified. Keep the
  Product Factory mapping as: no-variation → internal `variant_id` maps to the
  `item_id` **and** Product Factory keeps its own sellable-unit id regardless of
  whether Shopee later exposes a hidden model.
- **All numeric caps** (max tiers, options/tier, models/item) — `UNVERIFIED`.

---

## 11. Category Contract

- `get_category(language)` returns the per-region tree; `category_recommend(item_name)`
  predicts. Method names corroborated (S12, S13). `SEARCH_INDEXED`, MEDIUM.
- **Leaf-only requirement: `UNRESOLVED`** (recommended/likely, not proven — do
  not state as OFFICIAL; Adversarial Test on §18). No `listing_allowed`-style
  flag seen — do not import one.
- Category status / availability / seller restrictions / migration behaviour —
  `UNVERIFIED`.

---

## 12. Attribute Contract

- **Resource: `get_attribute_tree(category_id, language)`** (S13) — **corrects**
  Phase 02.1's `get_attributes`. `get_recommend_attribute(category_id, item_name)`
  is a **separate** resource for the recommended (quality) set.
- Field names (`is_mandatory`, `input_type`, `attribute_value_list`,
  `value_unit`, …) are from Phase 01 SDKs and are **`STILL_UNVERIFIED`** — the
  `get_attribute_tree` schema was not read.
- Conditional / shop-specific requiredness — no mechanism seen. `UNVERIFIED`.
- `get_cert_rule` suggests category-linked certification requirements surface via
  a dedicated resource (regulatory) — `SEARCH_INDEXED`, worth verifying for
  `compliance.md`.

---

## 13. Brand Contract

- `get_brand_list(category_id, offset, page_size, status)` — per-category, paged,
  with a `status` filter. `SEARCH_INDEXED`, MEDIUM.
- **`register_brand`** endpoint name now exists (S12) — Phase 01 said "not
  confirmed / maybe Seller-Center-only". Move to `SEARCH_INDEXED`; approval
  behaviour, "auto-revert to Sem marca", rejection reasons stay `UNVERIFIED`.
- **Brand requiredness is category-linked** (`get_brand_list` is per
  `category_id`) — this supports modelling brand as
  `CONDITIONAL_REQUIRED` / `PUBLICATION_REQUIRED` resolved per category, **not**
  `CORE_REQUIRED` (Adversarial Test H / §23). No evidence found that brand is
  *universally* mandatory in BR — keep that claim `⚠ verify`.
- "Sem marca" as the first list entry — still `SEARCH_INDEXED` (S2, Phase 01).

---

## 14. Identifier Contract

No new evidence. GTIN/EAN/UPC/ISBN handling, item-vs-model scope, per-category
requiredness, format validation, and absence handling all remain `UNVERIFIED`.
**No `EMPTY_GTIN_REASON` analogue seen** — do not create one. Never invent a code.

---

## 15. Numeric Limits

**Nothing locked. The dynamic-limit *source* is now itself in question.**

| Limit | Phase 02.1 value | Phase 02.2 result | Source | Verification | Stable/Dynamic |
|---|---|---|---|---|---|
| Title max/min | ≈ 255/256 / ≈ 10–25 | `STILL_UNVERIFIED` | unknown — **not** confirmed to come from `get_item_limit` | `SEARCH_INDEXED` (value) | DYNAMIC (source TBD) |
| Description max | ≈ 5,000 | `STILL_UNVERIFIED` | unknown | `SEARCH_INDEXED` | DYNAMIC (source TBD) |
| Image count | 1–9 | `STILL_UNVERIFIED` | unknown (S2 seller-edu, not API-confirmed) | `SEARCH_INDEXED` | DYNAMIC (source TBD) |
| Image dims / bytes | ≈350² / ≈1024² / ≈2 MB | `STILL_UNVERIFIED` | unknown | `SEARCH_INDEXED`/`OTHER_MARKET_REFERENCE` | DYNAMIC |
| Tier count | 2 | `STILL_UNVERIFIED` | unknown | `SEARCH_INDEXED` | DYNAMIC |
| Options/tier, models/item | — | `STILL_UNVERIFIED` | unknown | `UNVERIFIED` | DYNAMIC |
| SKU length | — | `STILL_UNVERIFIED` | — | `UNVERIFIED` | DYNAMIC |
| Price bounds | `price_limit` | `STILL_UNVERIFIED` | unknown (maybe `get_item_limit`, maybe category) | `UNVERIFIED` | DYNAMIC |
| Stock bounds | — | `STILL_UNVERIFIED` | — | `UNVERIFIED` | DYNAMIC |
| Days-to-ship | 7–30 (pre-order) | `STILL_UNVERIFIED` | logistics service, **not** `get_dts_limit` under product | `SEARCH_INDEXED` | DYNAMIC |
| Video duration | 60 s (Shopee Video) | `STILL_UNVERIFIED` for the **listing** video | — | `UNVERIFIED` | scope-separated |

**`get_item_limit` — scope conflict (Adversarial Test C + D):** S13 says the
method takes **no parameters** and returns "the item listing limit **for your
shop**" — i.e. a **listing-count quota**, not per-field size limits. S12 shows it
with `item_name`, `category_id`. These disagree. **We can no longer assert that
title/description/image limits come from `get_item_limit`.** Until primary docs
are read:

- keep every numeric limit `DYNAMIC` + provisional (unchanged), **but**
- change the *source* of `resolve_title_limit` / `resolve_description_limit` /
  `resolve_image_limits` / `resolve_variation_limits` / `resolve_price_bounds`
  from "`get_item_limit`" to **"resolution source `UNVERIFIED` — likely a mix of
  `get_item_limit` (quota), category metadata, and static seller-education rules;
  confirm in Phase 02.3/02.4"**.

This is a **CORRECTED** architecture point, not a removed one — the checks stay;
their cited mechanism becomes honest.

---

## 16. Image Contract

- Upload is a separate service: `media_space/upload_image` → `image_id` (persist
  the id; CDN URLs expire). `SEARCH_INDEXED` (S12).
- Item image field believed `image { image_id_list }`; ordering / cover / count
  / tier-option image mapping — `UNVERIFIED`.
- 1–9 / 1:1 / 3:4 / ≥60% / dims — all still seller-education-only, `SEARCH_INDEXED`,
  **not** API-confirmed, **not** locked.

---

## 17. Listing Video Contract

- `media_space/upload_video` (+ `get_video_upload_result`) → `video_upload_id`;
  believed one listing video per item. `SEARCH_INDEXED` (S12).
- Listing-video duration / aspect / size / moderation — `UNVERIFIED`. Shopee
  Video (60 s / 9:16) and Shopee Live rules remain **scope-separated** and must
  not be applied to the listing video (unchanged).

---

## 18. Price Contract

- `update_price(item_id, price_list)` — batch-shaped; `price_list` entries carry
  `model_id` when models exist. Item-level price for no-variation items.
  Currency BRL. `SEARCH_INDEXED`, MEDIUM.
- `original_price` vs promo/`current_price`; price-range display when models
  differ — `SEARCH_INDEXED` (Phase 01). Min/max, model-to-model gap, promo
  mechanics — `UNVERIFIED`. Promotions are a separate service (`promotion.go` in
  S12) — out of scope.

---

## 19. Stock Contract

- `update_stock(item_id, stock_list)` — batch-shaped; `stock_list` entries carry
  `model_id` when models exist. `SEARCH_INDEXED`, MEDIUM.
- Absolute vs incremental write; `seller_stock` / `shop_stock` split; reserved
  vs available — **`UNVERIFIED`**. Assume absolute-set per model as the Phase
  02.1 safe default; keep `resolve_inventory_model` pending.

---

## 20. Warehouse / Location Contract

- **No location dimension was observed** in `update_stock` triangulation — but
  absence in a summary is **not** proof it doesn't exist (Adversarial Test I).
- The Brazil `br` module concerns **enrollment / fiscal / block-status**, not
  multi-warehouse stock.
- **State: `inventory object partially verified; warehouse topology UNRESOLVED`.**
  Do **not** copy Mercado Livre Multi Origem. Keep `inventory.md` conservative.

---

## 21. Logistics Contract

- Separate `logistics` service (`get_channel_list`, `get_address`, and — expected
  — days-to-ship). Method names beyond `get_channel_list` / `get_address` are
  `SEARCH_INDEXED` at best. `weight` / `dimension` / `logistic_info` are
  item-level fields on `add_item` (`UNVERIFIED` requiredness).
- `get_weight_rec` (product service) offers a weight recommendation.
- Separate seller-config (addresses, enabled channels) from per-listing payload
  (weight, dims, chosen channels, DTS). Unchanged from Phase 02.1.

---

## 22. Create Item Contract (concept only)

```
add_item  (→ returns item_id)
  category_id      (from get_category / category_recommend, validated leaf)
  item_name        (≤ unverified limit)
  description       (≤ unverified limit; no contact/diversion content)
  price            (item-level; BRL)
  stock            (item-level)
  images           (image_id_list from media_space/upload_image)   — requiredness UNVERIFIED
  brand            ({brand_id, original_brand_name})               — requiredness category-linked
  attribute_list   (from get_attribute_tree; ids/values UNVERIFIED)
  weight, dimension, logistic_info, condition, item_sku, pre_order — all UNVERIFIED requiredness
      │
      ▼
init_tier_variation | add_model   (→ returns model_id[])
  tier_variation + model_list  (per-model tier_index, model_sku, price, stock)
```

Only `category_id`, `item_name`, `description`, `price`, `stock` are
corroborated-required (S13). Everything else is concept, not schema.

---

## 23. Validation / Error Contract

- **`NO_DEDICATED_VALIDATOR_FOUND`** — searched a 380-endpoint SDK and a
  100%-coverage SDK; there is **no** pre-publication `validate` / `dry-run` /
  `precheck` in `v2.product.*`. (Still not a *primary-confirmed* negative — phrase
  as "no dedicated validator endpoint found".)
- **`add_item` / `update_item` response is the gate.** Its shape (warnings array,
  failure list, `request_id`, field-level errors) is `UNVERIFIED`.
- **Post-creation diagnostics DO exist** (new this phase):
  `get_item_content_diagnosis_result(item_id)` and
  `get_item_list_by_content_diagnosis(diagnosis_status, …)` — a listing-quality
  diagnostic keyed by `item_id`, i.e. the closest thing to Mercado Livre's
  `/performance`. Feeds `QUALITY_STATUS`, **not** a pre-publication gate. BR
  availability `UNVERIFIED`.
- `get_item_violation_info(item_id)` — per-item policy-violation readout. Feeds
  compliance / `EXECUTION` post-publication. BR availability `UNVERIFIED`.

---

## 24. Item Lifecycle

- `unlist_item(item_id, unlist)` corroborated — the list/unlist toggle.
- `item_status` enum (`NORMAL` / `UNLIST` / `BANNED` / `DELETED` / `REVIEWING?`)
  — **not** re-verified this phase. `get_item_list` takes an `item_status`
  filter, which implies a small enum, but the values remain Phase 01 SDK
  reconstructions. `STILL_UNVERIFIED`. Keep the four provisional values; keep
  `REVIEWING` flagged `?`.
- Whether edits re-trigger review — `UNVERIFIED`.

---

## 25. Operation Table

| Operation | Resource | IDs required (input) | Produces | Readiness dimension |
|---|---|---|---|---|
| AUTHORIZE_SHOP | `auth/token/get`, `auth/access_token/get` | `partner_id`, `code` / `refresh_token`, `shop_id` | `access_token`, `refresh_token` | EXECUTION |
| GET_CATEGORIES | `product/get_category` | shop, `language` | category tree | PUBLICATION (feeds) |
| PREDICT_CATEGORY | `product/category_recommend` | `item_name` | ranked `category_id`s | — (discovery) |
| GET_ATTRIBUTES | `product/get_attribute_tree` | `category_id`, `language` | attribute set | PUBLICATION (feeds) |
| GET_RECOMMENDED_ATTRS | `product/get_recommend_attribute` | `category_id`, `item_name` | recommended set | QUALITY (feeds) |
| GET_BRANDS | `product/get_brand_list` | `category_id`, paging, `status` | brands | PUBLICATION (feeds) |
| GET_SHOP_LISTING_LIMIT | `product/get_item_limit` | shop (params disputed) | listing quota (±) | PUBLICATION / EXECUTION (feeds) |
| UPLOAD_MEDIA | `media_space/upload_image` / `upload_video` | file | `image_id` / `video_upload_id` | EXECUTION |
| CREATE_ITEM | `product/add_item` | none (category/title/desc/price/stock in body) | **`item_id`** | EXECUTION (must not require `item_id`) |
| INIT_VARIATION | `product/init_tier_variation` | `item_id`, `tier_variation`, `model` | `model_id[]` | EXECUTION |
| ADD_MODELS | `product/add_model` | `item_id`, `model_list` | `model_id[]` | EXECUTION |
| UPDATE_ITEM | `product/update_item` | `item_id` | — | EXECUTION |
| UPDATE_MODEL | `product/update_model` | `item_id`, `model_id` | — | EXECUTION |
| UPDATE_PRICE | `product/update_price` | `item_id` (+ `model_id` per entry) | — | EXECUTION |
| UPDATE_STOCK | `product/update_stock` | `item_id` (+ `model_id` per entry) | — | EXECUTION |
| UNLIST_ITEM | `product/unlist_item` | `item_id`, `unlist` | — | EXECUTION |
| DELETE_ITEM | `product/delete_item` | `item_id` | — | EXECUTION |
| GET_ITEM | `product/get_item_base_info` | `item_id_list` | item data | EXECUTION |
| GET_ITEM_VIOLATIONS | `product/get_item_violation_info` | `item_id` | violation list | COMPLIANCE / EXECUTION |
| GET_CONTENT_DIAGNOSIS | `product/get_item_content_diagnosis_result` | `item_id` | diagnosis | QUALITY |

All rows `SEARCH_INDEXED`. No row is primary-confirmed.

---

## 26. Execution Readiness

Unchanged in principle; refined by §25:

- **`CREATE_ITEM`** prerequisites: valid shop token + scope; a validated leaf
  `category_id`; assembled body. **Must not require `item_id` / `model_id`.**
  (Adversarial Test J satisfied.)
- **`INIT_VARIATION` / `ADD_MODELS`** require the `item_id` from `CREATE_ITEM`.
- **`UPDATE_PRICE` / `UPDATE_STOCK`** require `item_id` and, per entry, the
  `model_id` when models exist — i.e. exactly the ids those APIs consume, no
  more. No universal "all mappings must exist" rule.
- **`resolve_open_platform_br_access`** stays a `DYNAMIC` account check →
  `REVIEW` while unresolved, never a global `FAIL`.

---

## 27. Dynamic Check Audit

| Check | Verdict | Note |
|---|---|---|
| `resolve_leaf_category` | **KEEP** | source `product/get_category` + `category_recommend` (corroborated); leaf-only rule still `⚠ verify` |
| `resolve_category_attributes` | **KEEP + RENAME source** | `get_attributes` → **`get_attribute_tree`**; field `is_mandatory` `⚠ verify` |
| `resolve_recommended_attributes` | **KEEP** | source now named: `product/get_recommend_attribute` |
| `resolve_brand_requirement` | **KEEP** | `get_brand_list` is per-`category_id` → brand requiredness is category-linked (supports CONDITIONAL, not CORE) |
| `resolve_brand_authorisation` | **KEEP** | still `UNVERIFIED`; `register_brand` name now known |
| `resolve_identifier_requirement` | **KEEP** | no new evidence; still `UNVERIFIED` |
| `resolve_title_limit` | **KEEP + CORRECT source** | source is **not** confirmed to be `get_item_limit`; mark "resolution source `UNVERIFIED`" |
| `resolve_description_limit` | **KEEP + CORRECT source** | same |
| `resolve_image_limits` | **KEEP + CORRECT source** | same; media via `media_space/upload_image` |
| `resolve_variation_limits` | **KEEP + CORRECT source** | same |
| `resolve_price_bounds` | **KEEP + CORRECT source** | `price_limit` source unconfirmed (item-limit vs category) |
| `resolve_dts_limit` | **KEEP + MOVE source** | not a `product` resource → logistics service (`UNVERIFIED`) |
| `resolve_logistics` | **KEEP** | `logistics/get_channel_list`, `get_address` corroborated |
| `resolve_condition_allowed` | **KEEP** | `UNVERIFIED` |
| `resolve_inventory_model` | **KEEP** | warehouse topology `UNRESOLVED`; `update_stock(stock_list)` shape corroborated |
| `resolve_shop_api_capability` | **KEEP** | auth model corroborated; `get_item_violation_info` can feed the "item BANNED" sub-check |
| `resolve_model_exists` | **KEEP** | `get_model_list(item_id)` corroborated |
| `resolve_compliance_applicability` | **KEEP** | `get_cert_rule` may be a real per-category certification source (`SEARCH_INDEXED`) |
| `resolve_contact_diversion_clean` | **KEEP** | INTERNAL payload scan; unchanged |
| `resolve_open_platform_br_access` | **KEEP** | BR Open Platform existence triangulated; eligibility `UNRESOLVED` |
| *(new)* `resolve_content_diagnosis` | **ADD (optional, QUALITY)** | `get_item_content_diagnosis_result(item_id)` — post-publication quality signal; never a publication gate |

No check points at a **fictional** resource after this audit — but several point
at resources whose **schema** is unverified, which is now stated explicitly.

---

## 28. Phase 02.1 Assumptions — CONFIRMED (to SEARCH_INDEXED, MEDIUM)

- v2 is the current product API; base `/api/v2`; hosts as listed.
- Auth: `partner_id`+`partner_key`, OAuth per shop → `access_token` (~4 h) +
  `refresh_token` (~30 d), HMAC-SHA256, timestamp in seconds.
- Entity spine **Shop → Item → Tier Variation → Model**; `item_id` / `model_id`
  Shopee-assigned; models created via `init_tier_variation` / `add_model`
  (separate from `add_item`).
- `update_price` / `update_stock` are batch-shaped and model-aware.
- Media upload is a separate service returning ids; persist ids not URLs.
- No dedicated pre-publication validator.
- `CREATE_ITEM` does not consume the `item_id` it produces.
- Mercado Livre concepts (`User Product`, `Family`, `PARENT_PK`/`CHILD_PK`,
  `Multi Origem`, `listing_allowed`, `EMPTY_GTIN_REASON`) still have **no** Shopee
  analogue in evidence — keep the "do not import" notes.

## 29. Phase 02.1 Assumptions — CORRECTED

| # | Was | Now |
|---|---|---|
| C1 | attributes via `get_attributes` | **`get_attribute_tree(category_id, language)`**; `get_recommend_attribute` for the recommended set |
| C2 | numeric limits resolve via `get_item_limit` | `get_item_limit` is (per the better SDK) a **shop listing-quota** call with no params; the **source of title/description/image/variation/price limits is `UNVERIFIED`** — checks kept, cited mechanism corrected |
| C3 | `get_dts_limit` is a `product` resource | not in `v2.product.*` — expected under a **`logistics`** service; `UNVERIFIED` |
| C4 | "no per-item violation API"; "no `/performance`-style API for BR" | `get_item_violation_info(item_id)` exists; **content-diagnosis** API exists (`get_item_content_diagnosis_result`, `get_item_list_by_content_diagnosis`) — both `SEARCH_INDEXED`, BR availability `UNVERIFIED` |
| C5 | brand registration "not confirmed" | `product/register_brand` endpoint name exists (`SEARCH_INDEXED`) |
| C6 | Brazil Open Platform access — "no evidence either way" | **existence well-triangulated** (`openplatform.shopee.com.br` host; `br` API module; BR seller-edu arts. 3445 + 27314); **eligibility still `UNRESOLVED`**, leaning approval-gated (`CONFIRMED_RESTRICTED` in spirit) |
| C7 | `item`/`product` naming loosely equivalent | sharpen: **persisted id = `item_id`**; read endpoints accept `item_id_list` (≤ 50) or `product_id` |

## 30. Remaining Unverified (top items)

1. Every request/response **schema** (`add_item`, `add_model`,
   `init_tier_variation`, `get_attribute_tree`, `get_item_limit`, …).
2. Every **numeric limit** and its **resolution source**.
3. **Auth signing** base-string exact composition/order.
4. **Brazil eligibility**: who may register a partner app; which API
   function-groups are granted; sandbox availability for BR; production-approval
   bar; account-type restrictions.
5. **Stock/warehouse** model for BR (single pool vs `location_id`; absolute vs
   delta).
6. **`item_status`** enum values and transition graph; whether edits re-trigger
   review; draft-via-API.
7. **Leaf-only** category requirement (proof).
8. **`get_item_limit`** true scope (quota vs field limits vs both).
9. Attribute **field names** and conditional-requiredness mechanism.
10. Identifier (GTIN/EAN) requiredness, scope, absence handling.
11. **Create/update error contract** shape.
12. Catalogue / "produto" grouping concept (still no evidence; still assume
    standalone).

## 31. Skill Files Changed

| File | Change type | What |
|---|---|---|
| `references/official-sources.md` | CORRECTED/EXPANDED | added S12–S17; added the `open.shopee.com/documents/v2/v2.product.<m>?module=89` locator pattern; API-source-registry columns per brief §43; kept "no `LIVE`" |
| `references/api-and-auth.md` | CORRECTED (major) | endpoint registry rebuilt from the corroborated v2 `product` method set + doc locators; `get_attributes`→`get_attribute_tree`; `get_item_limit` scope caveat; `get_dts_limit` moved to logistics/unverified; added `register_brand`, `get_recommend_attribute`, `get_item_violation_info`, content-diagnosis, `get_cert_rule`, size-chart, kit; BR host `openplatform.shopee.com.br`; `br` module; dynamic-check registry sources updated; BR-access section updated (still `UNRESOLVED`) |
| `references/attributes.md` | CORRECTED | resource is `get_attribute_tree`; `get_recommend_attribute` for recommended set; field names `⚠ verify` |
| `references/brand-and-identifiers.md` | CORRECTED | `register_brand` endpoint name now `SEARCH_INDEXED`; brand requiredness is category-linked → CONDITIONAL, not universal |
| `references/product-structure.md` | CORRECTED | persisted id = `item_id`; read endpoints take `item_id_list` (≤50)/`product_id`; doc-locator note |
| `references/variations.md` | CONFIRMED (notes) | `init_tier_variation`/`add_model`/`update_model`/`delete_model`/`get_model_list` corroborated MEDIUM; `delete_model` takes `model_id`; models are a separate call; kit/bundle concept noted as unmodelled |
| `references/moderation-and-enforcement.md` | CORRECTED | `get_item_violation_info(item_id)` name now known (`SEARCH_INDEXED`); `get_item_list_by_content_diagnosis` |
| `references/quality-audit.md` | CORRECTED | content-diagnosis API is the candidate `/performance` analogue (`SEARCH_INDEXED`, BR `UNVERIFIED`); still not a pre-publication gate; add optional `resolve_content_diagnosis` |
| `references/pricing.md` | CONFIRMED (notes) | `update_price(item_id, price_list)` batch/model-aware corroborated |
| `references/inventory.md` | CONFIRMED (notes) | `update_stock(item_id, stock_list)` corroborated; warehouse topology still `UNRESOLVED` |
| `references/logistics.md` | CORRECTED | DTS/limits live in a `logistics` service, not `product`; `get_weight_rec` noted |
| `references/categories.md` | CONFIRMED (notes) | `get_category(language)` / `category_recommend(item_name)` corroborated; leaf-only still `⚠ verify` |
| `SKILL.md` | CORRECTED | §13 gaps G1–G3 refined; §8/§10 pointers; link to this report; `last_reviewed` bump |

No file was touched that this research did not produce evidence about.

## 32. Risks

| Risk | Severity | Mitigation |
|---|---|---|
| Portal + archive + `developer.shopee.com` all tool-blocked; SPA defeats proxying | HIGH | Phase 02.3 = **Primary Documentation Ingestion** from user-supplied authenticated exports, or MCP/sandbox calls |
| Locking limits on `get_item_limit` when it may be a quota resource | MEDIUM | done — checks kept, cited source demoted to `UNVERIFIED` |
| Treating corroborated community-SDK names as OFFICIAL | MEDIUM | all rows stay `SEARCH_INDEXED`; `⚠ verify`; authority ≠ verification |
| Brazil eligibility assumed open | MEDIUM | kept `UNRESOLVED`; `EXECUTION` stays `REVIEW` + manual fallback |
| `global_product` vs `product` confusion for a future CB seller | LOW-MEDIUM | scoped to `v2.product.*`; `global_product` explicitly out of scope |

## 33. Recommended Phase 02.3

**`Phase 02.3 — Primary Documentation Ingestion`.**

The blocker is not the *shape* of the contract (triangulation got us a credible
v2 `product` map) — it is that **no primary page can be read** in this
environment. Phase 02.3 should:

1. Have a maintainer with an `open.shopee.com` developer account **export**
   (PDF / HTML / screenshots) the pages behind these locators:
   `v2.product.add_item`, `v2.product.get_attribute_tree`,
   `v2.product.get_item_limit`, `v2.product.init_tier_variation`,
   `v2.product.add_model`, `v2.product.update_stock`,
   `v2.product.get_item_base_info`, plus `v2.auth.*`, the `logistics` service,
   and the **Brazil onboarding / region / app-eligibility** pages.
2. Provide the BR seller-education bodies for arts. **3445** and **27314**.
3. Ingest those into `official-sources.md` / `api-and-auth.md`, flipping rows to
   `LIVE` and locking only what the primary text states.
4. Then run **`Phase 02.4 — Numeric Limits & Category/Attribute/Brand rule
   locking`** — not before.

If BR eligibility turns out to be the hard blocker (no partner app possible),
branch to **`Phase 02.3 — Brazil Partner / API Eligibility Resolution`** instead
and keep the Skill's output publish-agnostic.

---

## Sources

All consulted 2026-08-28. No source read `LIVE`.

- `open.shopee.com` / `developer.shopee.com` — Open Platform (Tier A) — **UNREACHABLE**
  (fetch-tool blocked; SPA returns title only, incl. via
  `open-shopee-com.translate.goog`).
- `web.archive.org` — **UNREACHABLE** (fetch-tool blocked).
- `github.com/QuoVadis86/shopee-sdk` — Go community SDK (Tier E) — `SEARCH_INDEXED`
  — endpoint names; region hosts incl. `openplatform.shopee.com.br`; `br.go`
  module (`query_shop_enrollment_status`, `query_shop_invoice_error`,
  `query_shop_block_status`, `query_sku_block_status`); sign base string
  `partner_id + api_path + timestamp + [access_token] + [shop_id]`.
- `github.com/congminh1254/shopee-sdk` — TS community SDK (Tier E) — `SEARCH_INDEXED`
  — `docs/managers/product.md` maps ~25 `v2.product.*` methods to paths + required
  params + `open.shopee.com/documents/v2/...?module=89` locators; `schemas/`
  holds `v2.product.*.json` and `v2.global_product.*.json`.
- `rollout.com` Shopee integration guide (Tier D) — `SEARCH_INDEXED` — `/api/v2`
  base; `/product/get_item_base_info` param `product_id`, response `item_id`;
  `/api/v2/auth/token/get`; token 4 h.
- `publicapis.io/shopee-api`, `api2cart.com/api-technology/shopee-api/` (Tier D) —
  `SEARCH_INDEXED` — `/api/v2` base; sandbox `partner.test-stable.shopeemobile.com`;
  `partner_id`+`partner_key`+`shop_id`+`access_token`; HMAC-SHA256; "Open Platform"
  vs "Seller API"; "route to the correct country endpoint".
- Shopee BR Centro de Educação do Vendedor — arts. **3445** ("Shopee Open API
  Platform | Passo a Passo de Solicitação"), **27314** ("Open Platform Shopee:
  Guia Prático de Integração") (Tier B) — `SEARCH_INDEXED` (title + snippet; body
  is SPA) — Brazil Open Platform application + integration processes exist.
- `geckoapi.com.br/blog/extrair-dados-shopee-api/` (Tier E, BR, 2026) —
  `SEARCH_INDEXED` — BR store operations use "Open Platform … Aplicação e
  autorização conforme o programa oficial".
- Phase 01 report `research/shopee-listing-skill/discovery-report.md`;
  Phase 02.1 Skill `.claude/skills/shopee-listing-best-practices/` — internal.
