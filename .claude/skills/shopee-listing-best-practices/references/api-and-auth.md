# API & auth — Open Platform (PROVISIONAL / mostly UNVERIFIED)

last_reviewed: 2026-08-28
phase_02_2_reviewed: 2026-08-30
volatile: true
scope_note: >-
  Everything here is `GLOBAL_API` reconstructed from community SDKs and
  integrator blogs (`SEARCH_INDEXED` for path names, `UNVERIFIED` for every
  schema, limit and behaviour). NONE of it has been confirmed against
  `open.shopee.com` — that portal and `web.archive.org` are blocked at the
  fetch-tool level and the portal is a client-rendered SPA (no body even via a
  translation proxy). Phase 02.2 raised the v2 `product` endpoint *names + paths
  + key params* to "corroborated `SEARCH_INDEXED`, MEDIUM" (two independent
  community SDKs, one mapping every method to an `open.shopee.com/documents/v2/`
  locator) and corrected several Phase 02.1 details — see
  `research/shopee-api-contract/phase-02.2-report.md`. Brazil Open Platform
  *existence* is now well-triangulated; Brazil *eligibility* is still
  `UNRESOLVED` (§4). Do not build a client, an auth flow, or a publish pipeline
  from this file.

## 1. Critical current limitation

> **No Shopee Open Platform API contract has been verified from a readable
> primary API source.**

Consequences:

- No endpoint path, request field, payload shape, response shape or numeric
  limit below is authoritative. Treat all as `⚠ verify`.
- Community SDK / integrator endpoint names do **not** upgrade a rule to
  `OFFICIAL`. They establish only that an endpoint *plausibly* exists.
- The Skill output is **publish-agnostic**: a listing draft + audit JSON. It does
  not assume an execution pipeline exists.
- If and when a maintainer gets portal or sandbox access (Phase 02.3 —
  Primary Documentation Ingestion), transcribe the real pages, fill the schemas,
  and flip each row's `verification` from `SEARCH_INDEXED` / `UNVERIFIED` to
  `LIVE`. The doc-locator column below (`open.shopee.com/documents/v2/v2.product.<method>?module=89&type=1`)
  is where to look; those URLs have **not** been read.

## 2. Auth model (reconstructed — `SEARCH_INDEXED`, `⚠ verify`)

| Element | Reconstructed value | Verification |
|---|---|---|
| Partner identity | `partner_id` + `partner_key` (a.k.a. `partner_secret`) | `SEARCH_INDEXED` |
| Shop authorisation | OAuth redirect (`shop/auth_partner`) → auth `code` → token exchange; one app authorises many shops | `SEARCH_INDEXED` |
| Access token | `access_token`, lifetime ≈ 4 h | `SEARCH_INDEXED`; timing `⚠ verify` |
| Refresh token | `refresh_token`, lifetime ≈ 30 d, rotates on use | `SEARCH_INDEXED`; timing `⚠ verify` |
| Token scoping | per `shop_id`; stored per shop; one app authorises many shops | `SEARCH_INDEXED`, MEDIUM (S12–S15) |
| Version prefix | `/api/v2` | `SEARCH_INDEXED`, MEDIUM (S13 schema names + S14 + S15) |
| Hosts | global `partner.shopeemobile.com`; sandbox `partner.test-stable.shopeemobile.com`; CN `openplatform.shopee.cn`; **Brazil `openplatform.shopee.com.br`** | `SEARCH_INDEXED` (BR host: S12 only — **corroborate**) |
| Token exchange / refresh | `POST /api/v2/auth/token/get` · `POST /api/v2/auth/access_token/get` | `SEARCH_INDEXED`, MEDIUM |
| Request signing | HMAC-SHA256 over a base string ≈ `partner_id + api_path + timestamp + [access_token] + [shop_id]`; timestamp in **seconds** | `SEARCH_INDEXED`; exact composition & param order `⚠ verify` — **do not implement** |
| Multi-shop grouping | `merchant_id` for SIP / cross-border flows; `main_account_id` at token exchange in some flows (`UNVERIFIED`) | `SEARCH_INDEXED` |
| Local vs global catalogue | `v2.product.*` = a shop's own catalogue (BR listings); `v2.global_product.*` = cross-border seller catalogue — **out of scope** unless a CB model is confirmed | `SEARCH_INDEXED` (S13) |
| Rate limits | exist; values unknown; back off on a rate-limit error | `UNVERIFIED` |

## 3. Endpoint registry — v2 `product` service (paths corroborated `SEARCH_INDEXED`, MEDIUM; **all schemas `UNVERIFIED`**)

Paths are `/api/v2/product/<method>` unless noted. "Doc locator" =
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
| `get_brand_list` | brands for a category (`brand_id`, names; "Sem marca" first) | `category_id`, `offset`, `page_size`, `status` | S2, S12, S13 |
| `register_brand` | submit a seller/manufacturer brand — **Phase 01 said "not confirmed"; a name now exists** | — | S12 |
| `get_item_limit` | **scope disputed** — S13: no params, returns the shop's item *listing quota*; S12: `item_name`, `category_id`. **Not confirmed to be the source of title/description/image size limits.** | (disputed) | S12, S13 |
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
| `get_item_base_info` | core listing fields | `item_id_list` (≤ 50) — also seen as `product_id` (S14) | S13, S14 |
| `get_item_extra_info` | sales / views / likes | `item_id_list` | S12, S13 |
| `get_item_list` | listing ids by `item_status` | `offset`, `page_size`, `item_status` | S13 |
| `search_item` | search shop listings | `offset`, `page_size` | S12, S13 |
| `get_item_violation_info` | **per-item policy violations** — Phase 01 said none existed | `item_id` | S12, S13 |
| `get_item_content_diagnosis_result` | **per-item content diagnosis** (listing-quality signal; closest thing to an ML `/performance` — post-creation, not a pre-publish gate) | `item_id` | S12, S13 |
| `get_item_list_by_content_diagnosis` | items filtered by diagnosis status | `diagnosis_status`, `offset`, `page_size` | S12, S13 |

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

Lifecycle: `add_item` first → then `init_tier_variation` **or** `add_model` to
create models (generally a **separate call** from item creation).

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
| days-to-ship limit | **expected here, not in `product`** — Phase 02.1's `get_dts_limit` under `product` is corrected; exact resource `UNVERIFIED` | — |

### Brazil-specific (`br` service — `SEARCH_INDEXED`, S12; BR-only)
| Method | Purpose | Evidence |
|---|---|---|
| `br/query_shop_enrollment_status` | whether a shop is enrolled in BR operations | S12 |
| `br/query_shop_invoice_error` | nota-fiscal / invoice errors | S12 |
| `br/query_shop_block_status` · `br/query_sku_block_status` | BR shop / SKU block status (enforcement) | S12 |

### Enforcement / validation
| Resource | Status |
|---|---|
| `public/get_shop_penalty` / account-health | `UNVERIFIED` for BR (other markets) |
| pre-publication validate / dry-run | **NO_DEDICATED_VALIDATOR_FOUND** — searched a 380-endpoint SDK + a "100% coverage" SDK; no `validate` / `dry-run` / `precheck` in `v2.product.*`. Not a primary-confirmed negative. The `add_item` / `update_item` response is the gate (shape `UNVERIFIED`). `get_item_content_diagnosis_result` is **post**-creation, not a pre-publish check. |

## 4. Brazil access (gap G2) — existence triangulated, eligibility `UNRESOLVED`

Phase 02.2 evidence that a Brazil Open Platform **exists** (all `SEARCH_INDEXED`):

- a dedicated Brazil region host `openplatform.shopee.com.br` (S12);
- a dedicated `br` API service (`query_shop_enrollment_status`,
  `query_shop_invoice_error`, `query_shop_block_status`,
  `query_sku_block_status` — S12) — BR is a first-class region;
- Shopee **Brasil** Centro de Educação do Vendedor articles **3445** ("Shopee
  Open API Platform | Passo a Passo de Solicitação" — application step-by-step)
  and **27314** ("Open Platform Shopee: Guia Prático de Integração") — titles /
  snippets only; bodies are SPA (S16);
- BR integrator framing: "Open Platform … Aplicação e autorização conforme o
  programa oficial" (S17).

**Still `UNRESOLVED`:** who may register a partner app for BR; which API
function-groups a BR partner is granted; sandbox availability for BR; the
production-approval bar; account-type restrictions. The eventual answer is more
likely `CONFIRMED_RESTRICTED` (approval-gated) than `CONFIRMED_UNAVAILABLE` — but
that is not yet confirmed.

Until resolved (Phase 02.3):

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

| Check | Why | Source (resource / doc) | Pending effect | Confirmed-incompatibility effect | Verification |
|---|---|---|---|---|---|
| `resolve_leaf_category` | listing must sit under a valid leaf for the real product | `category_recommend` + `get_category` | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` | `SEARCH_INDEXED` |
| `resolve_category_attributes` | which attributes are mandatory for the category | **`get_attribute_tree`** (field `is_mandatory` `⚠ verify`) | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` (mandatory unmet) | `SEARCH_INDEXED` |
| `resolve_recommended_attributes` | completeness / exposure beyond the minimum | **`get_recommend_attribute`** (`category_id`, `item_name`) | `QUALITY = REVIEW` | — (never `FAIL`) | `SEARCH_INDEXED` |
| `resolve_brand_requirement` | is the brand attribute required for this category; is the value resolvable | `get_brand_list` (per `category_id`) + policy | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` if required + unset | `SEARCH_INDEXED` |
| `resolve_brand_authorisation` | IP-gated categories needing brand authorisation | `get_category` / `get_cert_rule` / policy | `PUBLICATION = REVIEW` | `PUBLICATION`/`EXECUTION = FAIL` if gated + no authorisation | `UNVERIFIED` |
| `resolve_identifier_requirement` | GTIN/EAN requiredness per category, at item or model level | `get_attribute_tree` / policy | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` if required + absent (no invented code) | `UNVERIFIED` |
| `resolve_title_limit` | max (and min) `item_name` length | **resolution source `UNVERIFIED`** — `get_item_limit` may be a shop listing *quota*, not field limits (report §15); likely category metadata + seller-education rules | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` (over a **resolved** limit) | `UNVERIFIED` |
| `resolve_description_limit` | max description length; `extended` availability | **resolution source `UNVERIFIED`** (as `resolve_title_limit`) | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` (over a **resolved** limit) | `UNVERIFIED` |
| `resolve_image_limits` | count, dimensions, ratio, byte cap, moderation | **resolution source `UNVERIFIED`** — upload via `media_space/upload_image`; limits likely `get_item_limit` (±) + category + seller-education | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` (0 compliant 1:1; over/under a **resolved** hard bound) | `UNVERIFIED` |
| `resolve_variation_limits` | max tiers / options per tier / models | **resolution source `UNVERIFIED`** (maybe `get_item_limit`, maybe category) | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` (exceeds a **resolved** cap) | `UNVERIFIED` |
| `resolve_price_bounds` | min/max price for the category | **resolution source `UNVERIFIED`** (`price_limit` — item-limit vs category unconfirmed) | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` (out of a **resolved** bound) | `UNVERIFIED` |
| `resolve_dts_limit` | days-to-ship range | **`logistics` service** (exact resource `UNVERIFIED`) — **not** a `product` resource | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` (out of a **resolved** range) | `UNVERIFIED` |
| `resolve_logistics` | ≥ 1 enabled channel; weight/dimensions present | `logistics/get_channel_list` | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` (none enabled / no weight) | `SEARCH_INDEXED` |
| `resolve_condition_allowed` | is `condition = USED` allowed for the category | `get_category` / policy | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` (USED not allowed) | `UNVERIFIED` |
| `resolve_inventory_model` | single seller pool vs multi-warehouse `location_id` for BR; absolute vs incremental writes | `get_item_base_info` / stock docs | `EXECUTION = REVIEW` | `EXECUTION = FAIL` if the op writes a stock shape BR rejects | `UNVERIFIED` |
| `resolve_shop_api_capability` | shop auth + token + scope valid for the target op; rate-limit headroom; item not `BANNED`/`DELETED` for an update | token store / `auth/*` / `get_item_base_info` / `get_item_violation_info` | `EXECUTION = REVIEW` | `EXECUTION = FAIL` (no valid token/scope; banned; rate-limited) | `SEARCH_INDEXED` |
| `resolve_model_exists` | a model-scoped op needs an existing `model_id` (never for CREATE) | `get_model_list` (`item_id`) | `EXECUTION = REVIEW` | `EXECUTION = FAIL` (no model) | `SEARCH_INDEXED` |
| `resolve_compliance_applicability` | prohibited / restricted / regulated status for this product/context | Shopee BR policy resolution; `get_cert_rule` (per-category certification — `⚠ verify`) | `PUBLICATION`/`EXECUTION = REVIEW` | `FAIL` per `compliance.md` | `SEARCH_INDEXED` |
| `resolve_contact_diversion_clean` | no contact / external-diversion strings in the assembled payload | payload scan (INTERNAL) | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` if still present at publish (`CONTENT` stays `PASS` if dropped) | `INTERNAL` |
| `resolve_open_platform_br_access` | is the Open Platform API usable for this BR shop at all (gap G2) | onboarding / partner support; `br/query_shop_enrollment_status` once authorised | `EXECUTION = REVIEW` | falls back to manual publish; not a product-truth `FAIL` | `UNVERIFIED` |
| `resolve_content_diagnosis` *(optional, post-publication)* | Shopee's own listing-quality diagnosis for an existing item | `get_item_content_diagnosis_result` (`item_id`) — BR availability `⚠ verify` | `QUALITY = REVIEW` | — (never `FAIL`) | `SEARCH_INDEXED` |

The same four readiness dimensions absorb every check — **no fifth dimension is
needed.**

## Sources

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
