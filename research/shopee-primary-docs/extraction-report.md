# Phase 02.3 — Extraction Report

date: 2026-08-30
branch: research/shopee-primary-docs
inputs: Phase 02.2 report + Correction 02.2A (the corroborated map); **zero
primary artifacts**.

---

## Executive checkpoint

**Update 2026-09-03 — artifacts received.** 35 official Shopee Open Platform
print-to-PDF exports were supplied and are now registered as `SPD-001…SPD-035`
(`evidence-registry.md`) and organized under
`docs/marketplaces/shopee/open-platform/`. Secret gate: **30 committed, 5 HELD**
(SPD-001 has an OAuth `code`+`shop_id` in an example URL; SPD-005–008 are
image-only §8-high-risk pages with no OCR available here).

**Extraction has NOT been performed.** Sections 1–18 below and the P0 completion
matrix still read as of the pre-artifact checkpoint except the "Primary artifact
obtained?" column (now ✓ where a `docs/` PDF exists). No claim in
`claim-registry.md` has moved to `PRIMARY_VERIFIED`; `reconciliation-report.md`
is unchanged. The next pass reads the PDFs and does field extraction +
reconciliation.

The original **`PRIMARY DOCUMENTATION REQUIRED`** decision (below) is
**superseded** — the blocker it described (no artifacts) is resolved.

---

## 1. Artifacts received

None. `evidence-registry.md` is empty.

## 2. Provenance validation

N/A — nothing to validate. When artifacts arrive: establish origin, title,
Shopee surface, market/region, API version, capture date, whether auth was
required, and completeness **from contents**, not file names, before extracting
any rule.

## 3. Brazil eligibility findings (P0)

None. Open: eligible seller/account types; approved-partner requirement; manual
approval; BR sandbox; granted API function-groups. See `claim-registry.md`
SCL-010…SCL-014. Highest priority — determines whether Product Factory can ever
execute against BR shops.

## 4. Auth findings (P0)

None. Open: authorization flow, token exchange/refresh, exact signing base
string, host per region, version. SCL-001…SCL-006.

## 5. Item findings (P0)

None. Open: `add_item` full required/conditional field set, response shape
(`item_id`, warnings, failures, `request_id`), `item_id_list` cap, `product_id`
scope, `item_sku` semantics. SCL-020…SCL-028.

## 6. Model / variation findings (P0)

None. Open: model lifecycle vs `add_item`; addressing; tier caps; **whether a
no-variation Item has a hidden/default Model** (SCL-033); post-sale mutability.
SCL-030…SCL-035.

## 7. Category findings (P0)

None. Open: leaf-only requirement (proof), category status/eligibility field,
seller restrictions, migration behaviour. SCL-040…SCL-043. Do **not** import
ML's `listing_allowed`.

## 8. Attribute findings (P0)

None. Open: whether `is_mandatory` is a real field (SCL-052); full attribute
schema; conditional/shop-specific requiredness; regulatory-attribute validation.
SCL-050…SCL-055.

## 9. Brand findings (P0)

None. Open: category-dependent requiredness (proof), "Sem marca" value,
`register_brand` behaviour, IP-gated categories. SCL-060…SCL-064.

## 10. Identifier findings (P0)

None. Open: GTIN/EAN API field, scope, per-category requiredness, format
validation, duplicate rules, absence mechanism. SCL-070…SCL-071. Never invent a
code or an absence mechanism.

## 11. Price findings (P0)

None. Open: item/model scope, batch shape, `price_limit` bounds + their real
source, promo-price separation. SCL-080…SCL-082.

## 12. Stock findings (P0)

None. Open: absolute vs delta write; `seller_stock`/`shop_stock`; available vs
reserved; **BR warehouse/`location_id` dimension** (SCL-093); concurrency/version
behaviour. SCL-090…SCL-094. Implementation-critical.

## 13. Logistics findings (P0)

**`LOGISTICS PRIMARY DOCUMENTATION MISSING`** — no `logistics`-service page
(`get_channel_list`, `get_address`, days-to-ship / handling-time limit) was among
the 35 supplied artifacts. This is a **P0 coverage gap**: it does not block
finishing the docs reorganization, but it **blocks declaring Phase 02.3 P0
contract coverage complete**. Open: DTS-limit resource (in the `logistics`
service per Phase 02.2, exact name unknown); weight/dimension requiredness;
pre-order; dangerous goods. SCL-100…SCL-102. Not to be solved in this
docs-safety correction.

## 14. Media findings (P1)

None. Open: `upload_image`/`upload_video` contract; image count/dimensions/ratio/
byte cap + item image field + tier-option mapping; listing-video constraints.
SCL-110…SCL-112.

## 15. Moderation / diagnosis findings (P1)

None. Open: whether `get_item_violation_info` / content-diagnosis are
post-publication enforcement vs quality; whether any can affect
publication/execution (default: **no** — QUALITY only). SCL-130…SCL-132.

## 16. Numeric values discovered

None. When a primary doc states a number, record: value · unit · resource ·
field · market · category-dependency · shop-dependency · API version · evidence
ID; then classify scope `GLOBAL_STATIC` / `MARKET_STATIC` / `CATEGORY_DYNAMIC` /
`SHOP_DYNAMIC` / `RESOURCE_DYNAMIC` / `UNKNOWN_SCOPE`. Do **not** turn a number
into a global Skill constant here — Phase 02.4 decides.

## 17. Errors / schema findings

None. Open: `add_item`/`update_item` error contract (error code, message,
`request_id`, field-level failure, warning array, failure list). SCL-023.

## 18. Remaining gaps

All Phase 02.2 gaps G1–G8 remain, plus everything in `claim-registry.md`
(SCL-001…SCL-142). Nothing was closed.

---

## P0 completion matrix (brief §44)

| Contract area | Primary artifact obtained? | Primary verification (extraction) | Remaining gap |
|---|---|---|---|
| Brazil eligibility | ⚠ partial — SPD-035 (BR SPI app creation); SPD-002/004/009/035 (journey) | not started | eligibility criteria, approval bar, function-groups — **SPD-001 (auth) + SPD-005–008 HELD** |
| Auth | ✓ SPD-001 *(HELD)* | not started | needs SPD-001 committed/reviewed; exact signing base string |
| Add Item | ✓ SPD-010 | not started | full field set, response/error shape |
| Get Item | ✓ SPD-011 | not started | `product_id` scope, status, field representation |
| Variations / Models | ✓ SPD-015–020 | not started | caps, **hidden default model** (SPD-020) |
| Categories | ✓ SPD-021, SPD-022 | not started | leaf-only rule, status field, restrictions |
| Attributes | ✓ SPD-023, SPD-024 | not started | `is_mandatory` reality, full schema, conditionality |
| Brands | ✓ SPD-025, SPD-026 | not started | category-dependent requiredness, `register_brand` behaviour |
| Identifiers | ⚠ likely inside SPD-010 / SPD-023 | not started | dedicated coverage; field, scope, requiredness, absence |
| Price | ✓ SPD-027 | not started | scope, batch, `price_limit` source |
| Stock | ✓ SPD-028 | not started | absolute/delta, warehouse dimension, concurrency |
| Logistics | ✗ no | none | no logistics/DTS page supplied — DTS resource, weight/dims requiredness, pre-order |

**Artifacts now cover ~11 / 12 P0 areas (logistics missing; auth + 4 onboarding
pages HELD). Extraction has not run — Phase 02.4 still cannot start.**

---

## Acquisition checklist (brief §46)

The maintainer needs an authenticated `open.shopee.com` developer account (and,
ideally, a BR sandbox partner app). For each item: export as PDF / HTML / print-
to-PDF / screenshot / API Explorer capture, **run a secret scan and redact**,
then add an `SPD-xxx` row to `evidence-registry.md`.

### P0 — Brazil eligibility / onboarding (highest priority)
- [ ] Brazil Open Platform **eligibility / application** page(s) — who can apply, account types, approval flow, production vs sandbox access, API permissions/function groups
- [ ] Shopee BR seller-education **art. 3445** — "Shopee Open API Platform | Passo a Passo de Solicitação" (full body; note if the id/title changed)
- [ ] Shopee BR seller-education **art. 27314** — "Open Platform Shopee: Guia Prático de Integração" (full body)
- [ ] Developer-console **app-creation / region-selection / authorization-region** screens (screenshots acceptable — mark `completeness = SCREENSHOT`)

### P0 — Auth
- [ ] Authorization + `auth/token/get` + `auth/access_token/get` reference pages
- [ ] Request-signing / base-string page (HMAC-SHA256, timestamp, param order)
- [ ] Host / API-version / rate-limit page

### P0 — Product operations (doc locators from Phase 02.2, `module=89`)
- [ ] `v2.product.add_item`
- [ ] `v2.product.update_item`
- [ ] `v2.product.get_item_base_info`
- [ ] `v2.product.init_tier_variation`
- [ ] `v2.product.add_model` (+ `update_model`, `delete_model`, `get_model_list`)
- [ ] `v2.product.get_category` (+ `category_recommend`)
- [ ] `v2.product.get_attribute_tree` (+ `get_recommend_attribute`)
- [ ] `v2.product.get_brand_list` (+ `register_brand`)
- [ ] `v2.product.get_item_limit`  ← **resolve exactly what it returns**
- [ ] `v2.product.update_price`
- [ ] `v2.product.update_stock` (+ any stock-read op needed to understand it)
- [ ] `v2.product.unlist_item` / `delete_item` / `item_status` values

### P0 — Logistics
- [ ] `logistics` service index (channel list, address, **days-to-ship limit** resource)

### P1 — after P0
- [ ] `media_space/upload_image` / `upload_video`
- [ ] `v2.product.get_item_violation_info`
- [ ] `v2.product.get_item_content_diagnosis_result` / `get_item_list_by_content_diagnosis`
- [ ] Anything titled validate / precheck / dry-run / listing diagnostic (validator search)

### Do NOT collect (out of scope unless a listing dependency forces it)
Orders, Returns, Ads, Affiliate, Chat, Finance.

---

## Adversarial tests (brief §49) — outcomes at this checkpoint

| Test | Situation | Required outcome | This checkpoint |
|---|---|---|---|
| A | community SDK says field X mandatory, primary silent | do not upgrade X | held — X stays `UNVERIFIED` (SCL-052 etc.) |
| B | primary global page confirms endpoint, BR eligibility page absent | endpoint `PRIMARY_VERIFIED`/`GLOBAL_API`, BR execution unresolved | N/A — no primary page yet |
| C | primary confirms numeric max 9, one resource/market | record with exact scope, not a global constant | N/A — no primary numbers |
| D | screenshot shows half a schema | only visible fields `PRIMARY_VERIFIED` | rule recorded for intake |
| E | old v1 page vs current v2 page conflict | `VERSION_DIFFERENCE`, use current | rule recorded for intake |
| F | primary confirms `get_item_limit` = shop listing quota | Phase 02.2 correction confirmed; never a title/image/variation limit source | pre-committed: SCL-120/121 stay this way unless a primary doc says otherwise |
| G | primary schema exposes hidden/default Model for no-variation item | correct the PF external mapping | pre-committed: SCL-033/141 open |
| H | primary docs show BR Product API needs approved-partner access | BR capability = conditional/account-specific, **not** globally unavailable; model as `resolve_open_platform_br_access` | pre-committed: SCL-012, brief §47 |
| I | official content-diagnosis API exists | post-publication QUALITY capability, not a pre-publication validator (unless docs say otherwise) | pre-committed: SCL-131 |
| J | **no primary artifact supplied** | return `PRIMARY DOCUMENTATION REQUIRED` + exact checklist; no web-derived fake completion | **this is the current state** |

---

## Final decision

```
PRIMARY DOCUMENTATION REQUIRED
```

No primary Shopee artifact was supplied or is reachable. Per the phase brief this
is a **legitimate checkpoint outcome**, not a failure: Phase 02.2 gave a credible
map; Phase 02.3 needs primary evidence that the map is correct, and that evidence
can only come from a maintainer with authenticated portal / sandbox access
supplying the artifacts in the acquisition checklist above.

**Do not** proceed to Phase 02.4 (Rule Locking). **Do not** run another broad web
research pass. **Do not** upgrade any claim from community/integrator sources.

Resume Phase 02.3 by supplying artifacts (see `README.md` §"How a maintainer
resumes this phase").
