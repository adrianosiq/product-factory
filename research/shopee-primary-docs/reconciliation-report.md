# Phase 02.3 — Reconciliation Report

date: 2026-08-30
compares: Phase 02.2 claims (`research/shopee-api-contract/phase-02.2-report.md`
+ Correction 02.2A) ⇄ ingested primary evidence.

**Status: no primary evidence ingested → every relevant claim reconciles as
`STILL_UNVERIFIED`.** This table becomes meaningful only after `SPD-xxx`
artifacts populate `evidence-registry.md`.

Classification vocabulary (brief §43): `PRIMARY_CONFIRMED` · `CORRECTED` ·
`PARTIALLY_CONFIRMED` · `STILL_UNVERIFIED` · `CONFLICTING` · `OUT_OF_SCOPE`.

## Reconciliation

| Phase 02.2 claim | Phase 02.2 state | Primary evidence | Classification | Note |
|---|---|---|---|---|
| Auth = OAuth per shop; token lifetimes; HMAC-SHA256 signing (exact base string unverified) | `SEARCH_INDEXED` · MULTI_SOURCE | (none) | `STILL_UNVERIFIED` | SCL-001…003 |
| `/api/v2` is the current-generation surface | `SEARCH_INDEXED` · MULTI_SOURCE | (none) | `STILL_UNVERIFIED` | SCL-004 |
| BR host `openplatform.shopee.com.br` (existence signal, SINGLE_SOURCE) | `SEARCH_INDEXED` · LOW | (none) | `STILL_UNVERIFIED` | SCL-006 — not confirmed as canonical Product API base |
| Brazil Open Platform application/onboarding process exists | `SEARCH_INDEXED` (article titles only) | (none) | `STILL_UNVERIFIED` | SCL-010 |
| Brazil eligibility / approved-partner requirement | `UNRESOLVED` | (none) | `STILL_UNVERIFIED` | SCL-011…014; may become `resolve_open_platform_br_access` conditional (brief §47) |
| `add_item` returns `item_id`; minimal fields category/name/description/price/stock | `SEARCH_INDEXED` · MULTI_SOURCE (return) / single-doc (fields) | (none) | `STILL_UNVERIFIED` | SCL-020…021 |
| Full `add_item` schema + response/error contract | `UNVERIFIED` | (none) | `STILL_UNVERIFIED` | SCL-022…023 |
| `get_item_base_info` takes `item_id_list` (≤ 50) | `SEARCH_INDEXED` · SINGLE_SOURCE | (none) | `STILL_UNVERIFIED` | SCL-024 |
| `product_id` is/ isn't a local alias for `item_id` | `UNRESOLVED` | (none) | `STILL_UNVERIFIED` | SCL-025 — possible `v2.global_product` / cross-border concept |
| `item_status` enum `NORMAL`/`UNLIST`/`BANNED`/`DELETED`(/`REVIEWING?`) | `STILL_UNVERIFIED` | (none) | `STILL_UNVERIFIED` | SCL-027 |
| Models created by a separate call after `add_item`; `add_model` → `model_id`s | `SEARCH_INDEXED` · MULTI_SOURCE | (none) | `STILL_UNVERIFIED` | SCL-030 |
| ≤ 2 tiers; positional options; `tier_index[]` | `SEARCH_INDEXED`; cap unverified | (none) | `STILL_UNVERIFIED` | SCL-032 |
| No-variation Item has a hidden/default Model — ? | `UNRESOLVED` | (none) | `STILL_UNVERIFIED` | SCL-033 — blocks final PF mapping (SCL-141) |
| `get_category(language)` / `category_recommend(item_name)` | `SEARCH_INDEXED` · MULTI_SOURCE | (none) | `STILL_UNVERIFIED` | SCL-040 |
| Leaf-only category requirement | `UNRESOLVED` (likely) | (none) | `STILL_UNVERIFIED` | SCL-041 |
| Attributes via `get_attribute_tree`; recommended via `get_recommend_attribute` | `SEARCH_INDEXED` (names) | (none) | `STILL_UNVERIFIED` | SCL-050…051 |
| `is_mandatory` is a real field | `STILL_UNVERIFIED` | (none) | `STILL_UNVERIFIED` | SCL-052 |
| Brands via per-category `get_brand_list`; requiredness category-dependent | `SEARCH_INDEXED` (inferred) | (none) | `STILL_UNVERIFIED` | SCL-060…061 |
| `register_brand` is an API resource | `SEARCH_INDEXED` (name observed) | (none) | `STILL_UNVERIFIED` | SCL-063 |
| GTIN/EAN field, scope, requiredness, absence mechanism | `UNVERIFIED` | (none) | `STILL_UNVERIFIED` | SCL-070…071 |
| `update_price(item_id, price_list)` batch/model-aware; BRL | `SEARCH_INDEXED` · MEDIUM | (none) | `STILL_UNVERIFIED` | SCL-080 |
| `price_limit` bounds + source | `UNVERIFIED` | (none) | `STILL_UNVERIFIED` | SCL-081 |
| `update_stock(item_id, stock_list)` batch/model-aware | `SEARCH_INDEXED` · MEDIUM | (none) | `STILL_UNVERIFIED` | SCL-090 |
| Stock absolute vs delta; BR warehouse/`location_id`; concurrency | `UNVERIFIED` / `UNRESOLVED` | (none) | `STILL_UNVERIFIED` | SCL-091…094 |
| `logistics/get_channel_list`, `get_address` | `SEARCH_INDEXED` | (none) | `STILL_UNVERIFIED` | SCL-100 |
| DTS-limit resource is in the `logistics` service, not `product` | `UNVERIFIED` (Phase 02.2 filing correction) | (none) | `STILL_UNVERIFIED` | SCL-101 — the correction stands pending a primary doc |
| `media_space/upload_image` → `image_id`; persist ids not URLs | `SEARCH_INDEXED` | (none) | `STILL_UNVERIFIED` | SCL-110 |
| Image count/dims/ratio (1–9 / 1:1 / 3:4 / ≈350²) | `UNVERIFIED` (seller-education only) | (none) | `STILL_UNVERIFIED` | SCL-111 |
| `get_item_limit` = shop listing quota, NOT the size-limit source | `SEARCH_INDEXED` (Phase 02.2 correction) | (none) | `STILL_UNVERIFIED` | SCL-120 — the correction stands; test F confirms it only if a primary doc does |
| Real source of title/description/image/variation/price limits | `UNVERIFIED` | (none) | `STILL_UNVERIFIED` | SCL-121 |
| No dedicated pre-publication validator | `NO_DEDICATED_VALIDATOR_FOUND` (absence not proven) | (none) | `STILL_UNVERIFIED` | SCL-124 — stays "not found", not "does not exist" |
| `get_item_violation_info` — post-publication enforcement | `SEARCH_INDEXED` (name observed) | (none) | `STILL_UNVERIFIED` | SCL-130 |
| Content-diagnosis API — post-publication QUALITY, not a gate | `SEARCH_INDEXED` (name observed) | (none) | `STILL_UNVERIFIED` | SCL-131 — routing to `QUALITY_STATUS` only stays the default |
| `CREATE_ITEM` does not consume the `item_id` it returns | `SEARCH_INDEXED` · MULTI_SOURCE (also a design invariant) | (none) | `PARTIALLY_CONFIRMED` (design invariant holds regardless; API shape unverified) | operation-scoped `EXECUTION_STATUS` unaffected |
| ML concepts (`User Product`, `Family`, `PARENT_PK`, `Multi Origem`, `listing_allowed`, `EMPTY_GTIN_REASON`, `/items/validate`, `/performance`) have no Shopee analogue | absence of evidence | (none) | `STILL_UNVERIFIED` (kept as "do not import" — not a positive claim) | unchanged |

## Contradictions with Phase 02.2

None — no primary evidence to contradict anything. When primary docs arrive and
disagree with Phase 02.2, record: previous claim · previous evidence · new
primary evidence · scope difference · resolution (`CORRECTED` /
`VERSION_DIFFERENCE` / `MARKET_DIFFERENCE` / `UNRESOLVED_CONFLICT`). Primary wins,
subject to version / market / date / scope. Keep historical evidence; do not
delete it.

## Readiness model

Unchanged. `CONTENT_STATUS` / `PUBLICATION_STATUS` / `EXECUTION_STATUS` /
`QUALITY_STATUS` are not redesigned. No fifth dimension. Primary evidence, when
ingested, may only refine which rules feed each dimension.

## Skill files updated this phase

**None.** No claim became `PRIMARY_VERIFIED` and nothing was directly corrected
by a primary artifact, so per brief §36 the Shopee Skill is untouched. The
`research/shopee-primary-docs/` intake structure is the entire deliverable.
