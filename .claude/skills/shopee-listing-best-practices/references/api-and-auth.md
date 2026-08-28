# API & auth — Open Platform (PROVISIONAL / mostly UNVERIFIED)

last_reviewed: 2026-08-28
volatile: true
scope_note: >-
  Everything here is `GLOBAL_API` reconstructed from community SDKs and
  integrator blogs (`SEARCH_INDEXED` for path names, `UNVERIFIED` for every
  schema, limit and behaviour). NONE of it has been confirmed against
  `open.shopee.com`. Whether the Open Platform API is available to Brazil
  partners/shops at all is an OPEN QUESTION (gap G2). Do not build a client, an
  auth flow, or a publish pipeline from this file.

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
- If and when a maintainer gets portal or sandbox access (Phase 02.2 / 02.3),
  transcribe the real pages, fill the schemas, and flip each row's
  `verification` from `SEARCH_INDEXED` / `UNVERIFIED` to `LIVE`.

## 2. Auth model (reconstructed — `SEARCH_INDEXED`, `⚠ verify`)

| Element | Reconstructed value | Verification |
|---|---|---|
| Partner identity | `partner_id` + `partner_key` (a.k.a. `partner_secret`) | `SEARCH_INDEXED` |
| Shop authorisation | OAuth redirect (`shop/auth_partner`) → auth `code` → token exchange; one app authorises many shops | `SEARCH_INDEXED` |
| Access token | `access_token`, lifetime ≈ 4 h | `SEARCH_INDEXED`; timing `⚠ verify` |
| Refresh token | `refresh_token`, lifetime ≈ 30 d, rotates on use | `SEARCH_INDEXED`; timing `⚠ verify` |
| Token scoping | per `shop_id`; stored per shop | `SEARCH_INDEXED` |
| Hosts | prod `partner.shopeemobile.com`; sandbox `partner.test-stable.shopeemobile.com` | `SEARCH_INDEXED` |
| Request signing | HMAC-SHA256 over a base string ≈ `partner_id + path + timestamp + access_token + shop_id`; timestamp in **seconds** | `SEARCH_INDEXED`; exact composition & param order `⚠ verify` |
| Multi-shop grouping | `merchant_id` for SIP / cross-border flows | `SEARCH_INDEXED` |
| Rate limits | assume conservative (order of ~10 rps / ~1000/min); back off on a rate-limit error | `UNVERIFIED` |

## 3. Endpoint registry (paths `SEARCH_INDEXED`; **all schemas `UNVERIFIED`**)

`v2` unless noted. Grouped by purpose. "Purpose" and "evidence" are what we
believe; do not code against these.

### Auth
| Resource | Purpose | Evidence | Verification |
|---|---|---|---|
| `shop/auth_partner` (redirect) | seller authorises the app for a shop → auth `code` | S9 | `SEARCH_INDEXED` |
| `auth/token/get` | exchange `code` → `access_token` + `refresh_token` | S8, S9 | `SEARCH_INDEXED` |
| `auth/access_token/get` | refresh the access token | S8, S9 | `SEARCH_INDEXED` |

### Category / attribute / brand / limits (all `DYNAMIC` sources — never hardcode their outputs)
| Resource | Purpose | Evidence | Verification |
|---|---|---|---|
| `product/category_recommend` | predict candidate categories from item name/image | S7, S9 | `SEARCH_INDEXED` |
| `product/get_category` | category tree for the region: `category_id`, `parent_category_id`, names, `has_children` | S7 | `SEARCH_INDEXED` |
| `product/get_attributes` | attributes for a category: `attribute_id`, `is_mandatory`, `input_type`, `format_type`, value list, units | S6, S7 | `SEARCH_INDEXED` |
| `product/get_brand_list` | brands for a category: `brand_id`, names; "Sem marca" as first option | S2, S7 | `SEARCH_INDEXED` |
| `product/get_item_limit` | numeric limits: name length, image count/size, price/stock bounds, tier/model caps | S7 | `SEARCH_INDEXED`; response shape `UNVERIFIED` |
| `product/get_dts_limit` | days-to-ship min/max for a category | S7 | `SEARCH_INDEXED` |

### Item lifecycle
| Resource | Purpose | Evidence | Verification |
|---|---|---|---|
| `product/add_item` | create a listing (full payload) | S7, S9 | `SEARCH_INDEXED`; schema `UNVERIFIED` |
| `product/update_item` | edit listing fields | S7, S9 | `SEARCH_INDEXED` |
| `product/update_item_sku` | set `item_sku` | S7 | `SEARCH_INDEXED` |
| `product/delete_item` | delete a listing | S7 | `SEARCH_INDEXED` |
| `product/unlist_item` | list/unlist toggle (batch) | S7 | `SEARCH_INDEXED` |
| `product/get_item_base_info` | core listing fields | S9 | `SEARCH_INDEXED` |
| `product/get_item_extra_info` | sales / views / likes | S7 | `SEARCH_INDEXED` |
| `product/get_item_list` | listing ids by `item_status` | S9 | `SEARCH_INDEXED` |
| `product/search_item` | search shop listings | S7 | `SEARCH_INDEXED` |

### Variation / model
| Resource | Purpose | Evidence | Verification |
|---|---|---|---|
| `product/init_tier_variation` | set the tier structure + initial models on a no-variation item | S7 | `SEARCH_INDEXED` |
| `product/update_tier_variation` | edit tier option names / images | S7 | `SEARCH_INDEXED` |
| `product/add_model` | add models (combinations) | S7 | `SEARCH_INDEXED` |
| `product/update_model` | edit a model (price / stock / sku) | S7 | `SEARCH_INDEXED` |
| `product/delete_model` | remove a model | S7 | `SEARCH_INDEXED` |
| `product/get_model_list` | list an item's models + tier structure | S7 | `SEARCH_INDEXED` |

### Price / stock
| Resource | Purpose | Evidence | Verification |
|---|---|---|---|
| `product/update_price` (+ batch) | price (item or per model) | S7 | `SEARCH_INDEXED` |
| `product/update_stock` (+ batch) | stock (item or per model); absolute-set assumed | S6, S7 | `SEARCH_INDEXED`; absolute-vs-incremental `⚠ verify` for BR |

### Media
| Resource | Purpose | Evidence | Verification |
|---|---|---|---|
| `media_space/upload_image` | upload image → `image_id` (CDN URLs expire — persist the id) | S9 | `SEARCH_INDEXED` |
| `media_space/upload_video` / `get_video_upload_result` | listing video → `video_upload_id` | S7 | `SEARCH_INDEXED`; constraints `UNVERIFIED` |

### Logistics
| Resource | Purpose | Evidence | Verification |
|---|---|---|---|
| `logistics/get_channel_list` | logistics channels enabled for the shop | S9 | `SEARCH_INDEXED` |
| `logistics/get_address` | pickup / return / default `address_id` | S9 | `SEARCH_INDEXED` |

### Enforcement (BR availability unconfirmed)
| Resource | Purpose | Evidence | Verification |
|---|---|---|---|
| `public/get_shop_penalty` / account-health | penalty points / shop health | other markets | `UNVERIFIED` for BR |
| pre-publication validate / dry-run | — | — | **NOT FOUND** — no `POST /items/validate` analogue confirmed (absence not proven — gap G5) |

## 4. Brazil access risk (gap G2)

Unknown whether a Brazil-domiciled partner / shop can obtain Open Platform API
access at all. Until resolved:

- Assume **manual / Seller-Center publishing**. The Skill produces a draft + audit
  for a human to enter, not an API payload to POST.
- `EXECUTION_STATUS` for any operation that needs the API stays `REVIEW`
  ("API availability for this BR shop not confirmed") — never `FAIL` on that
  basis alone.
- Do not design queues, token stores, retry logic or a publish pipeline in this
  phase.

Resolve via Shopee Open Platform onboarding for a BR shop / Shopee partner
support (Phase 02.3).

## 5. Dynamic check registry

Every `DYNAMIC` decision the Skill makes is one of these checks. **No endpoint is
hardcoded** — each names a resource whose own `verification` state is in §3.
`pending` always → `REVIEW` (never `FAIL`); only an **executed** check that
**confirms a mandatory incompatibility** → `FAIL`.

| Check | Why | Source (resource / doc) | Pending effect | Confirmed-incompatibility effect | Verification |
|---|---|---|---|---|---|
| `resolve_leaf_category` | listing must sit under a valid leaf for the real product | `category_recommend` + `get_category` | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` | `SEARCH_INDEXED` |
| `resolve_category_attributes` | which attributes are mandatory for the category | `get_attributes` (`is_mandatory`) | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` (mandatory unmet) | `SEARCH_INDEXED` |
| `resolve_recommended_attributes` | completeness / exposure beyond the minimum | `get_attributes` (recommended set) | `QUALITY = REVIEW` | — (never `FAIL`) | `SEARCH_INDEXED` |
| `resolve_brand_requirement` | is the brand attribute required for this category; is the value resolvable | `get_brand_list` + policy | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` if required + unset | `SEARCH_INDEXED` |
| `resolve_brand_authorisation` | IP-gated categories needing brand authorisation | `get_category` / policy | `PUBLICATION = REVIEW` | `PUBLICATION`/`EXECUTION = FAIL` if gated + no authorisation | `UNVERIFIED` |
| `resolve_identifier_requirement` | GTIN/EAN requiredness per category, at item or model level | `get_attributes` / policy | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` if required + absent (no invented code) | `UNVERIFIED` |
| `resolve_title_limit` | max (and min) `item_name` length | `get_item_limit` | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` (over limit) | `SEARCH_INDEXED` |
| `resolve_description_limit` | max description length; `extended` availability | `get_item_limit` / live docs | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` (over limit) | `UNVERIFIED` |
| `resolve_image_limits` | count, dimensions, ratio, byte cap, moderation | `get_item_limit` + image-rules doc | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` (0 compliant 1:1; over/under a hard bound) | `SEARCH_INDEXED` (count) / `UNVERIFIED` (dims) |
| `resolve_variation_limits` | max tiers / options per tier / models | `get_item_limit` | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` (exceeds cap) | `UNVERIFIED` |
| `resolve_price_bounds` | min/max price for the category | `get_item_limit` (`price_limit`) | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` (out of bounds) | `UNVERIFIED` |
| `resolve_dts_limit` | days-to-ship range for the category | `get_dts_limit` / `get_category` | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` (out of range) | `SEARCH_INDEXED` |
| `resolve_logistics` | ≥ 1 enabled channel; weight/dimensions present | `logistics/get_channel_list` | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` (none enabled / no weight) | `SEARCH_INDEXED` |
| `resolve_condition_allowed` | is `condition = USED` allowed for the category | `get_category` / policy | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` (USED not allowed) | `UNVERIFIED` |
| `resolve_inventory_model` | single seller pool vs multi-warehouse `location_id` for BR; absolute vs incremental writes | `get_item_base_info` / stock docs | `EXECUTION = REVIEW` | `EXECUTION = FAIL` if the op writes a stock shape BR rejects | `UNVERIFIED` |
| `resolve_shop_api_capability` | shop auth + token + scope valid for the target op; rate-limit headroom; item not `BANNED`/`DELETED` for an update | token store / `auth/*` / `get_item_base_info` | `EXECUTION = REVIEW` | `EXECUTION = FAIL` (no valid token/scope; banned; rate-limited) | `SEARCH_INDEXED` |
| `resolve_model_exists` | a model-scoped op needs an existing `model_id` (never for CREATE) | `get_model_list` | `EXECUTION = REVIEW` | `EXECUTION = FAIL` (no model) | `SEARCH_INDEXED` |
| `resolve_compliance_applicability` | prohibited / restricted / regulated status for this product/context | Shopee BR policy resolution | `PUBLICATION`/`EXECUTION = REVIEW` | `FAIL` per `compliance.md` | `SEARCH_INDEXED` |
| `resolve_contact_diversion_clean` | no contact / external-diversion strings in the assembled payload | payload scan (INTERNAL) | `PUBLICATION = REVIEW` | `PUBLICATION = FAIL` if still present at publish (`CONTENT` stays `PASS` if dropped) | `INTERNAL` |
| `resolve_open_platform_br_access` | is the Open Platform API usable for this BR shop at all (gap G2) | onboarding / partner support | `EXECUTION = REVIEW` | falls back to manual publish; not a product-truth `FAIL` | `UNVERIFIED` |

The same four readiness dimensions absorb every check — **no fifth dimension is
needed.**

## Sources

- Community SDKs — `github.com/wjp-letgo/shopeego` (v2 names), `github.com/teacat/shopeego`
  (v1 field names; `tier_index` semantics; pre-order 7–30), `github.com/raviMukti/shopee-api-client`,
  `github.com/mu-hanz/shoapi` (v2 hosts; token 4 h / refresh 30 d) — external —
  consulted 2026-08-28 — `SEARCH_INDEXED` — endpoint names, host URLs, auth
  timing outline. Not a canonical contract.
- Integrator guides — `developer.inlinex.com.sg/blog/shopee-api-integration-guide-sellers`,
  `rollout.com`, `api2cart.com` — external — consulted 2026-08-28 —
  `SEARCH_INDEXED` — v2 paths; sign base string; timestamp in seconds; "store
  image ids not URLs".
- Shopee Open Platform API portal — `https://open.shopee.com` — Open Platform —
  consulted 2026-08-28 — `UNVERIFIED` (UNREACHABLE) — the source that must
  replace every row above.
- `public/get_shop_penalty` / account-health — other Shopee markets — consulted
  2026-08-28 — `UNVERIFIED` for BR.
- Full API table: `research/shopee-listing-skill/discovery-report.md` §"API Table",
  §"Dynamic Check Table", §29 (U1, U2, U13, U16).
