# Shopee Listing Best Practices — Discovery Report

**Phase 01 — Discovery + Source Mapping + Architecture. Not an implementation.**

research_date: 2026-08-28
market_scope: Shopee **Brasil** (site/region BR). Other Shopee markets are used
only to understand the Open Platform's shape and are labelled
`OTHER MARKET — REFERENCE ONLY`.
author: product-create team

---

## 0. How to read this report

- **Nothing here is a Skill rule.** It is a research map.
- Rule classification tags are the Product Factory set (`OFFICIAL`, `DYNAMIC`,
  `INTERNAL`, `EXPERIMENTAL`, `LEARNED`, `UNVERIFIED`) — §8 evaluates whether that
  set is still sufficient for Shopee (conclusion: yes).
- **Verification quality** is recorded separately from classification, per the
  brief §7: `LIVE` (page read directly), `SEARCH_INDEXED` (reconstructed from
  search snippets / third-party / SDKs), `UNVERIFIED` (insufficient evidence).
- **There are zero `LIVE` Shopee sources in this report.** `open.shopee.com` is
  unreachable from the research environment; `seller.shopee.com.br/edu`,
  `help.shopee.com.br` and `ads.shopee.com.br/learn` render client-side and
  return only their page title to a fetch. Everything Shopee-official is
  `SEARCH_INDEXED`; all API field/limit detail is `UNVERIFIED`. This is the same
  posture the Mercado Livre Skill records as `⚠ verify`.
- Where a number is given with "≈" it is a `SEARCH_INDEXED` value of MEDIUM/LOW
  confidence and **must not be hardcoded** — it is a `DYNAMIC` value to resolve
  from `get_item_limit` / live docs.

---

## 1. Executive Summary

**What Shopee Brasil's model looks like (best current understanding).**

- The **shop** (`shop_id`, Shopee-assigned, region BR) is the seller storefront.
  A partner **app** (`partner_id`) is authorised per shop by OAuth.
- The **item** (`item_id`, Shopee-assigned) *is the listing*. It carries the
  title (`item_name`), description, one leaf `category_id`, a **mandatory brand
  attribute** (Brasil-specific — see §16), category attributes, 1–9 images
  (1:1 mandatory), weight, package dimensions, logistics channels, `condition`,
  pre-order / days-to-ship, and a seller-controlled `item_sku`.
- The **model** (`model_id`, Shopee-assigned) is the *sellable variant*. Shopee
  is a **combination / matrix** model: with tiers `Cor{Preto,Branco}` ×
  `Tamanho{P,M,G}` Shopee creates **6 models**, each = one combination
  (`tier_index` array), each with its own `model_sku` (seller-controlled), price
  and stock. This maps cleanly to Product Factory `variant_id` / SKU **at the
  model level**.
- **tier_variation** holds the axes (max **2 tiers** — corroborated across SDKs;
  ⚠ verify the cap and per-tier option/model caps for BR). Tier-1 options can
  each carry an image.
- Shopee has **no** User Product, no Family, no PARENT_PK/CHILD_PK, no Multi
  Origem, no `listing_allowed`, no `EMPTY_GTIN_REASON`, no shared catalog product
  page / Buy Box, no `/items/validate`, no `/performance`, no
  `moderations/last_moderation`. Those are Mercado Livre concepts. Do not port
  them. (Whether Shopee BR has *any* catalogue/"produto" grouping concept is
  **UNRESOLVED** — §3.6.)

**What transfers from the Mercado Livre work.**

The four-dimension readiness model (`CONTENT` / `PUBLICATION` / `EXECUTION` /
`QUALITY`), operation-scoped `EXECUTION`, the pending-vs-FAIL safety rule, the
evidence model, requirement layering, Product Identity Guard, claim-safety,
the compliance-as-resolution-procedure model, return-prevention's
"materially-different-product" test, and the internal-id ↔ external-id mapping
discipline **all appear to transfer** and are marked `SHARED_CORE_CANDIDATE`
(§26). None should be *extracted* yet — two marketplaces is not enough.

**Blocking gaps** (§30, §31): (a) API portal unreadable → every endpoint/limit is
`SEARCH_INDEXED`/`UNVERIFIED`; (b) **whether the Open Platform API is available
to Brazil-domiciled partners/shops at all is UNRESOLVED** — the whole `EXECUTION`
layer depends on it; (c) core numeric limits only MEDIUM/LOW confidence;
(d) BR stock/warehouse model (single pool vs multi-warehouse `location_id`)
unresolved; (e) no pre-publication validator; (f) catalogue/"produto" concept
unresolved.

**Decision: DISCOVERY GAPS MUST BE RESOLVED FIRST** (§65).

---

## 2. Source Quality

### 2.1 Source map

| # | Source | Title / area | Market | Level | Verification | Notes / rules supported |
|---|---|---|---|---|---|---|
| S1 | `open.shopee.com` — Shopee Open Platform API docs | v2 API: product / model / tier_variation / category / attribute / brand / logistics / media / auth | Global API | 1 | **UNREACHABLE** (fetch blocked; not indexed usefully) | Nothing read. All v2 facts below are reconstructed. |
| S2 | `seller.shopee.com.br/edu` — Centro de Educação do Vendedor | listing rules, images, titles, brand, prohibited products, restricted-item release | BR | 2 | `SEARCH_INDEXED` (page = title only on fetch; body via snippets) | image count 1–9, 1:1 mandatory / 3:4 recommended, cover-photo text ban, ≥60% frame logo rule, Title Case guidance, brand mandatory + "Sem marca", restricted-item authorisation flow |
| S3 | `help.shopee.com.br` — Central de Ajuda | Política de Produtos Proibidos e Restritos (art. 76226); Shopee Video/Live policies; afiliados | BR | 2 | `SEARCH_INDEXED` | prohibited vs restricted split; regulators ANVISA/ANATEL/INMETRO/MAPA/ANS; homologation-absent → prohibited |
| S4 | `ads.shopee.com.br/learn` — Shopee Ads education | product-page quality, creative guidelines, video duration | BR | 3 | `SEARCH_INDEXED` | listing-quality *recommendations* only; not organic requirements; not proven ranking factors |
| S5 | `deo.shopeemobile.com/.../Pontos de Penalidade.pdf` — official Shopee BR PDF | penalty-point table | BR | 2 | `SEARCH_INDEXED` (PDF referenced, not parsed) | weekly accrual, 60-day validity, per-infraction points, progressive sanctions |
| S6 | GitHub `teacat/shopeego` (Go) | request/response structs | Global API | 4 | `SEARCH_INDEXED` | **v1** API — field *names* only; limits stale, do not trust numbers |
| S7 | GitHub `wjp-letgo/shopeego/product` (Go) | v2 product package function list | Global API | 4 | `SEARCH_INDEXED` | v2 endpoint names: AddItem/UpdateItem/UpdateItemSku/AddModel/UpdateModel/GetModelList/InitTierVariation/UpdateTierVariation/UpdatePrice/UpdateStock/GetCategory/GetAttributes/CategoryRecommend/GetBrandList/GetItemLimit/GetDtsLimit/SearchItem/UnlistItem; `item_status` ∈ NORMAL/BANNED/UNLIST/DELETED |
| S8 | GitHub `raviMukti/shopee-api-client` (PHP), `mu-hanz/shoapi` (PHP) | v2 client | Global API | 4 | `SEARCH_INDEXED` | host `partner.shopeemobile.com` (prod) / `partner.test-stable.shopeemobile.com` (sandbox); token 4 h; refresh 30 d |
| S9 | `developer.inlinex.com.sg`, `rollout.com`, `api2cart.com` — integrator guides | v2 endpoint paths, auth, signing | Global API | 4 | `SEARCH_INDEXED` | `/api/v2/auth/token/get`, `/api/v2/product/add_item`, `/api/v2/product/get_item_base_info`, `/api/v2/product/update_item`, `/api/v2/media_space/upload_image`; sign base `partner_id+path+timestamp+access_token+shop_id` HMAC-SHA256; timestamp in seconds |
| S10 | `base.com`, `anymarket`, `ideris`, `maino`, `gobots`, `destraveescale`, `1001clicks`, `mambadigital` — BR integrators / educators | Shopee BR listing constraints, penalty system, brand registration, 2026 image-rule update | BR | 4 | `SEARCH_INDEXED` | title ≈ 255–256 chars; description ≈ 5,000 chars; image min ≈ 350×350, recommended ≈ 1024×1024; EAN "obrigatório para alguns produtos"; brand mandatory attribute; 2026 stricter image moderation |
| S11 | `blog.gs1br.org` | GTIN/EAN on Shopee BR | BR | 4 | `SEARCH_INDEXED` | EAN presented as best practice, **not** a blanket listing requirement |

### 2.2 What this means for the Skill

- The Mercado Livre Skill could at least corroborate OFFICIAL rules against
  search-indexed copies of the real developer pages. For Shopee, the developer
  portal is not even indexed in a usable way — **the API contract must be
  treated as unverified until a maintainer with portal access transcribes it**,
  or until the MCP/API layer can call the endpoints and read the responses.
- Brazil-official policy (S2, S3, S5) is reconstructable but only in outline.
- Every rule the Skill emits in Phase 02 starts life tagged `⚠ verify`.

---

## 3. Shopee Product / Listing Model

### 3.1 The hierarchy (proposed, `SEARCH_INDEXED` + `UNVERIFIED` on detail)

```
shop            shop_id       Shopee-assigned, per region (BR)      — the storefront
  └─ item       item_id       Shopee-assigned                       — THE LISTING
       │        item_sku      seller-controlled, optional, mutable  — seller identity
       ├─ tier_variation      seller-defined axes, ≤2 tiers         — the variation structure
       │        (no id; positional; option_list per tier; tier-1 options may hold an image)
       └─ model  model_id     Shopee-assigned                       — THE SELLABLE VARIANT
                model_sku     seller-controlled, optional, mutable  — seller variant identity
                tier_index[]  which option of each tier this model is  (e.g. [0,2] = tier1 opt0 + tier2 opt2)
                price, stock  per model
```

Terminology note: v2 endpoints mix **"item"** and **"product"** — e.g.
`get_item_base_info` takes `item_id_list`, but some integrator docs call the
argument `product_id`. Treat `item` and `product` as the **same listing entity**
in v2 unless the portal proves otherwise. `model` is the v2 word; older v1
material says `variation`.

### 3.2 Answers to the brief's §9 questions

| Question | Best current answer | Verification |
|---|---|---|
| What represents the listing? | **item** (`item_id`) | `SEARCH_INDEXED` |
| What represents the sellable variant? | **model** (`model_id`); if the item has no variation, the item itself is the sellable unit | `SEARCH_INDEXED` |
| What represents seller-controlled identity? | `item_sku` (listing) and `model_sku` (variant) — free-text, seller-set, mutable, mostly buyer-invisible | `SEARCH_INDEXED`; optionality & visibility `UNVERIFIED` |
| Which IDs are Shopee-assigned? | `shop_id`, `item_id`, `model_id` (+ `brand_id`, `category_id`, `attribute_id`, `value_id`, `logistics_channel_id`, image/media ids) | `SEARCH_INDEXED` |
| Which identifiers must Product Factory persist? | internal `variant_id` ↔ (`shop_id`, `item_id`, `model_id`); and the internal SKU stored *into* `item_sku` / `model_sku`. Persist the media ids too (Shopee CDN URLs expire — store `image_id`, not URL). | `INTERNAL` design |

### 3.3 Item-level fields (reconstructed — names `SEARCH_INDEXED`, requiredness `UNVERIFIED`)

`item_name`, `description` (+ `description_type` `normal`|`extended` and an
`extended_description` block of image+text — availability for BR `UNVERIFIED`),
`category_id`, `brand` (`{ brand_id, original_brand_name }`), `attribute_list`
(`[{ attribute_id, attribute_value_list | original_value_name }]`), `image`
(`{ image_id_list }`), `video_upload_id` (listing video — `UNVERIFIED`),
`weight` (kg), `dimension` (`{ package_length, package_width, package_height }`
cm), `logistic_info` (`[{ logistic_id, enabled, is_free, size_id?, ... }]`),
`condition` (`NEW` | `USED`), `pre_order` (`{ is_pre_order, days_to_ship }`),
`item_sku`, `item_status`, `seller_stock` / `stock` (see §14), `price` (see §13),
`tax_info` / fiscal fields (BR — `UNVERIFIED`), `wholesale` tiers,
`item_dangerous` / dangerous-goods flag (`UNVERIFIED`).

### 3.4 What is NOT present (checked, not assumed)

- **No User Product** — no ML-style "what is sold" entity separate from the
  listing. The model *is* the sellable unit and it belongs to one item.
- **No Family / family_name** — the item's own `item_name` is the only title;
  there is no generated family name across listings.
- **No PARENT_PK / CHILD_PK** — tier axes are seller-declared and positional,
  not derived from attribute metadata tags.
- **No `listing_allowed` category flag** found — category eligibility appears to
  be enforced implicitly (leaf required; restricted categories gated by
  authorisation). ⚠ verify whether `get_category` exposes an eligibility flag.
- **No `EMPTY_GTIN_REASON`** — no structured "legitimately has no barcode"
  mechanism found. GTIN is just an optional/category-conditional field.

### 3.5 Catalogue / shared product page

**UNRESOLVED.** No evidence of an ML-style `catalog_product_id` shared page with
a Buy Box. Shopee BR has a "Configuração do Produto" / "Guia de Categorias" tool
and there is v2 "product" vocabulary, but nothing found indicates multiple
sellers competing on one canonical product node. Safe assumption for Phase 02:
**every Shopee listing is standalone**; do not build catalogue-association logic
until this is confirmed either way.

### 3.6 Listing lifecycle / status — see §17.

---

## 4. Category Model

| Aspect | Finding | Tag | Verification |
|---|---|---|---|
| Tree | `get_category` (v2) → `category_id`, `parent_category_id`, `original_category_name`, `display_category_name`, `has_children` | OFFICIAL | `SEARCH_INDEXED` |
| Leaf required | Must list under a leaf (`has_children = false`) | OFFICIAL | `SEARCH_INDEXED` (strongly implied) |
| Prediction | `category_recommend` (v2) → ranked `category_id` list from item name/image | OFFICIAL | `SEARCH_INDEXED` |
| Site scope | Category tree is **per region** (BR tree); a `language` / region param is passed. Not shop-specific. | OFFICIAL | `SEARCH_INDEXED` |
| Category drives | mandatory attributes, whether brand is required, `days_to_ship` limits (`get_dts_limit` / `DaysToShipLimits.{min,max}`), size-chart support, allowed logistics, allowed `condition`, prohibited/restricted status | OFFICIAL | `SEARCH_INDEXED` |
| Change after publish | possible via `update_item`; may invalidate attributes; some categories restricted; ⚠ verify | OFFICIAL | `UNVERIFIED` |
| Availability flag | ⚠ verify whether an explicit eligibility/`status` field exists on a category | — | `UNVERIFIED` |

**Safe flow** (matches the Mercado Livre lesson — predictor is discovery, not
authority): `category_recommend` → choose candidate leaf → `get_category`
confirms it is a leaf and correct → `get_attributes` + `get_brand_list` +
`get_dts_limit` for that leaf. `discover → resolve → validate` is adoptable.

---

## 5. Attribute Model

| Aspect | Finding | Tag | Verification |
|---|---|---|---|
| Endpoint | `get_attributes` (v2), per `category_id` | OFFICIAL | `SEARCH_INDEXED` |
| Fields | `attribute_id`, `original_attribute_name`, `is_mandatory` (bool), `input_type` (`DROP_DOWN` / `MULTIPLE_SELECT` / `TEXT_FILLING` / `COMBO_BOX` / …), `format_type` (`NORMAL` / `QUANTITATIVE`), `attribute_value_list` (`value_id`, `original_value_name`, `value_unit`), `date_format` | OFFICIAL | `SEARCH_INDEXED`; exact enum set `UNVERIFIED` |
| Requiredness | **static per category**, expressed by `is_mandatory`. No separate "conditional_required" resolution endpoint (no analogue of ML's `POST .../attributes/conditional`) was found. | OFFICIAL | `SEARCH_INDEXED` |
| Conditional / shop-specific requiredness | **UNRESOLVED** — some categories surface "recommended" attributes for quality; whether requiredness ever depends on other field values or on the shop is unconfirmed | — | `UNVERIFIED` |
| Free-text vs value-id | both, per `input_type`; `TEXT_FILLING` allows free text, dropdowns need `value_id` | OFFICIAL | `SEARCH_INDEXED` |
| Units | `value_unit` on quantitative attributes | OFFICIAL | `SEARCH_INDEXED` |
| Regulatory fields | appear as **ordinary attributes** in regulated categories (e.g. INMETRO registration number, ANVISA registration) — not a separate subsystem | OFFICIAL | `SEARCH_INDEXED` |

Requiredness classification for the Skill: **static + category-specific**
(`DYNAMIC` — always fetch `get_attributes`; never hardcode which attributes a
category needs). Treat "recommended attributes" as `QUALITY`, not
`PUBLICATION` (same split the Mercado Livre Skill makes for
`technical_specs/input`).

---

## 6. Variation / Model Architecture

### 6.1 Structure

- **Tiers:** 0, 1, or 2. (Cap of 2 corroborated across SDKs and BR integrator
  docs; ⚠ verify for BR.)
- **Options per tier:** a list of option names; tier-1 options may each carry an
  image. v1 limited an option name to ~20 chars — **`UNVERIFIED` for v2/BR**.
- **Model = combination.** `init_tier_variation` / `add_model` take a
  `model_list` where each model has `tier_index` (e.g. `[0,1]` = tier-1 option 0
  + tier-2 option 1), `model_sku`, `original_price`, `seller_stock`/`stock`.
  With 2 tiers of 2 and 3 options → **6 models** (full matrix). Whether Shopee
  auto-creates the full matrix or only the models you submit is **`UNVERIFIED`**
  (v1 required you to submit each; assume the same).
- **Max models:** `UNVERIFIED` (SEA historically ~50; ⚠ verify for BR via
  `get_item_limit`).
- **Post-sale mutability:** `init_tier_variation` "cannot edit an existing
  tier_variation"; structural change after sales is restricted. `update_model` /
  `update_tier_variation` handle prices/stock/option edits within limits.
  Exact rules `UNVERIFIED`.
- **Naming restrictions:** no special characters / no trailing spaces / not
  misleading (from BR title guidance, likely applies to option names) —
  `UNVERIFIED`.

### 6.2 API surface (v2, names `SEARCH_INDEXED`)

`init_tier_variation`, `update_tier_variation`, `add_model`, `update_model`,
`delete_model`, `get_model_list`, `update_price`, `update_stock`
(+ batch variants).

### 6.3 Mapping to Product Factory

The **model** is the Product Factory `variant_id` / SKU anchor. Persist
`model_id ↔ variant_id`; write the internal SKU into `model_sku`. The
`tier_index` + option names give the human-readable axis values but are **not**
identity. A no-variation item maps its single sellable unit to the `item_id`
(and `item_sku`).

---

## 7. SKU & External IDs

| Field | Level | Assigned by | Stable? | Buyer-visible? | Persist? |
|---|---|---|---|---|---|
| `shop_id` | shop | Shopee | yes | no | yes (account mapping) |
| `item_id` | listing | Shopee | yes (life of listing) | in URL | yes ↔ internal product/listing ref |
| `model_id` | variant | Shopee | yes (life of model) | no | yes ↔ internal `variant_id` |
| `item_sku` | listing | **seller** | mutable | mostly no (⚠ verify) | store = internal SKU |
| `model_sku` | variant | **seller** | mutable | mostly no (⚠ verify) | store = internal variant SKU |
| `brand_id` | attribute | Shopee (or seller-registered → Shopee-approved) | yes | yes (as name) | yes |
| media `image_id` / `video_upload_id` | media | Shopee | yes; **URLs expire** | yes | yes (store id, not URL) |

Rules: SKU is (apparently) **optional to publish** and **not unique-enforced by
Shopee** (⚠ verify) — so Product Factory must enforce its own SKU uniqueness and
never rely on Shopee to dedupe. Never use `item_name` / option text as canonical
identity when `item_id` / `model_id` / SKU exist.

---

## 8. Rule classification — is the Product Factory set still enough?

**Yes.** `OFFICIAL` / `DYNAMIC` / `INTERNAL` / `EXPERIMENTAL` / `LEARNED` /
`UNVERIFIED` cover every Shopee finding in this report. Notes:

- `DYNAMIC` will carry more weight than in Mercado Livre: almost every numeric
  limit (title, image dims, description, tier/option/model caps, price/stock
  bounds, days-to-ship) is a `get_item_limit` / `get_dts_limit` / category
  lookup and **must not be a constant**.
- `LEARNED` maps naturally to Shopee's **penalty-point** feedback and shop
  performance metrics.
- Do **not** add a Shopee-only class. The "surface scope" of a rule
  (listing vs Shopee Video vs Shopee Live vs Ads) is important (§24, §46) but is
  a *field on the rule*, not a new classification tag.

---

## 9. Title

| Rule | Value | Tag | Verification |
|---|---|---|---|
| Max length | ≈ **255–256** characters (BR) | DYNAMIC (resolve `get_item_limit`) | `SEARCH_INDEXED`, MEDIUM. **Not** the ~60/120 of some older SEA markets. |
| Min length | ≈ 10–25 chars (varies by source) | DYNAMIC | `UNVERIFIED` |
| Prohibited chars | no special characters, no trailing/leading spaces | OFFICIAL | `SEARCH_INDEXED` |
| Case | Title Case (capitalise each word); avoid ALL CAPS | OFFICIAL (recommendation) | `SEARCH_INDEXED` |
| Content policy | no seller contact/social, no external diversion, no misleading/inaccurate keywords, no keyword spam, no unrelated brand names | OFFICIAL | `SEARCH_INDEXED` |
| Promo terms | avoid "Frete Grátis", "Promoção", "Desconto 50%" etc. in the title | OFFICIAL | `SEARCH_INDEXED` |
| Recommended shape | concise + informative; include brand, product line, model, distinguishing specs (material, main ingredient, colour, size, quantity) *when available* | OFFICIAL (recommendation) | `SEARCH_INDEXED` |
| "Produto + Marca + Características + Modelo" template | — | INTERNAL / third-party | `SEARCH_INDEXED` — a convention, not a Shopee rule |
| "First ≈65 chars are what shows before the fold" | — | EXPERIMENTAL / LEARNED | `SEARCH_INDEXED` (third-party) — **not** an official rule, **not** a stated ranking factor |

Keep the Mercado Livre discipline: separate **hard constraint** (length, chars)
from **official content policy** (no diversion, no misleading) from **official
recommendation** (shape) from **internal optimisation** from **SEO hypothesis**.

---

## 10. Search / SEO

Shopee officially says (S2/S4, `SEARCH_INDEXED`) that listing **completeness and
quality** — title, images, description, correct category, product code (EAN),
and attributes — plus **relevance, conversion/sales, ratings, seller
performance / penalty points, price competitiveness, shipping (free shipping,
Shopee Express), and Ads** — influence exposure. "Well-filled listings get more
exposure" is stated directly.

| Signal group | Examples | Tag |
|---|---|---|
| OFFICIAL (stated to affect exposure) | listing completeness, category correctness, attribute fill, image compliance, relevance, sales/conversion, ratings, penalty points, shipping options | OFFICIAL (SEARCH_INDEXED) |
| LEARNED (from our data, future) | which completeness elements move impressions for our catalogue | LEARNED |
| EXPERIMENTAL (hypothesis) | title keyword ordering, first-N-chars weighting, 3:4-image ranking lift, attribute-count thresholds | EXPERIMENTAL |

Do **not** reverse-engineer the ranking algorithm in Phase 02. "Shopee
recommends X" must not silently become "X is a ranking factor" (brief §19).

---

## 11. Description

| Rule | Value | Tag | Verification |
|---|---|---|---|
| Long-description limit | ≈ **5,000** characters (BR) | DYNAMIC | `SEARCH_INDEXED`, MEDIUM |
| Format | plain text baseline; `description_type = extended` gives an image+text block layout in some flows | OFFICIAL | availability for BR `UNVERIFIED` |
| Prohibited | external links, phone numbers, QR codes, social handles, "chame no WhatsApp", any off-Shopee diversion; seller contact info; misleading/inaccurate claims | OFFICIAL | `SEARCH_INDEXED` |
| Recommended content | dimensions, material, usage, what's in the box, warranty/return terms; structured and scannable | OFFICIAL (recommendation) | `SEARCH_INDEXED` |
| API version differences | `normal` vs `extended` description bodies differ; ⚠ verify which BR + which categories support `extended` | — | `UNVERIFIED` |

Contact-info / external-diversion strings are **removable content-policy
violations** — same handling the Mercado Livre Skill uses: drop the string,
`CONTENT_STATUS` can stay `PASS`, `PUBLICATION_STATUS = FAIL` only if the
assembled payload still carries it at publish.

---

## 12. Images

| Rule | Value | Tag | Verification |
|---|---|---|---|
| Count | **1–9** images | OFFICIAL | `SEARCH_INDEXED`, HIGH (consistent) |
| Aspect ratio | **1:1 mandatory** for product images; **3:4** optional and recommended for extra search exposure | OFFICIAL | `SEARCH_INDEXED`, MEDIUM-HIGH |
| Min dimensions | stated variously ≈ **350×350** (min resolution) to 500×500 / 800×800 | DYNAMIC (`get_item_limit`) | `SEARCH_INDEXED`, LOW-MEDIUM — do not hardcode |
| Recommended dimensions | ≈ **1024×1024** (1:1); some sources ≈ 1200×1200 | INTERNAL/recommendation | `SEARCH_INDEXED`, LOW |
| File types | JPG / PNG (likely JPEG/PNG) | OFFICIAL | `UNVERIFIED` exact set |
| Size cap | ≈ 2 MB/image (SEA docs) | DYNAMIC | `UNVERIFIED` for BR |
| Cover / main image | white background (recommended→required, varies by source); **no** commercial text ("Frete Grátis", "50% Off", "Super Promoção"); no watermark; no border; no logo/mascot; no Shopee-brand elements; no sensitive content | OFFICIAL | `SEARCH_INDEXED` |
| Product-as-logo | if a product photo is used as the shop/brand logo, the product must fill **≥ 60%** of the frame, centered | OFFICIAL | `SEARCH_INDEXED` |
| Image per variation | tier-1 options may each carry one image | OFFICIAL | `SEARCH_INDEXED` |
| Moderation | automated + manual; 2026 rules tightened — non-compliant images → listing banned/hidden | OFFICIAL | `SEARCH_INDEXED` |

Keep the Mercado Livre three-layer separation: **(A)** OFFICIAL Shopee specs and
moderation, **(B)** INTERNAL gallery strategy (how many *useful* images, roles),
**(C)** Product Identity Guard (§23). Creative presets (85% fill, tilt angles)
are `INTERNAL`, never presented as Shopee policy.

---

## 13. Pricing

| Aspect | Finding | Tag | Verification |
|---|---|---|---|
| Level | model-level when models exist, else item-level | OFFICIAL | `SEARCH_INDEXED` |
| Fields | `original_price` and promo/`current_price`; currency **BRL** | OFFICIAL | `SEARCH_INDEXED` |
| Update | `update_price` + batch; `update_model` for model price | OFFICIAL | `SEARCH_INDEXED` |
| Bounds | `get_item_limit.price_limit` (min/max) | DYNAMIC | `UNVERIFIED` values |
| Range display | item shows a price range when models differ | OFFICIAL | `SEARCH_INDEXED` |
| Misleading pricing | Shopee BR polices fake "de/por" reference prices | OFFICIAL | `SEARCH_INDEXED`, specifics `UNVERIFIED` |
| Cross-variant consistency | no rule found that all models must be same price; kit/pack differences allowed | — | `UNVERIFIED` |

**Classification for Product Factory:** price is **Commercial context** +
**Publication requirement** (must be present and within `price_limit` to
publish) + **Execution state** (an `UPDATE_PRICE` op). It is **not**
ProductMaster truth. Same stance as the Mercado Livre Skill.

---

## 14. Inventory

| Aspect | Finding | Tag | Verification |
|---|---|---|---|
| Level | model-level when models exist, else item-level | OFFICIAL | `SEARCH_INDEXED` |
| Update | `update_stock` + batch; `update_model` for model stock | OFFICIAL | `SEARCH_INDEXED` |
| Structure | v2 has evolved toward `stock_info_v2` with `seller_stock` (and a Shopee-side `shop_stock` reflecting fulfilment inventory) | OFFICIAL | `SEARCH_INDEXED`, shape `UNVERIFIED` |
| Multi-warehouse | `seller_stock[]` can carry a `location_id` in **some** regions | OFFICIAL (other markets) | **UNRESOLVED for BR** |
| Absolute vs incremental | `update_stock` sets an absolute value (SEA docs) | OFFICIAL | `UNVERIFIED` for BR |
| Reserved / available | Shopee tracks reserved vs sellable internally; seller writes the sellable figure | OFFICIAL | `SEARCH_INDEXED` |
| Concurrency / versioning | no optimistic-lock token found; last-write-wins assumed | — | `UNVERIFIED` |
| Fulfilment stock | inventory in Shopee's DCs / "Envios Shopee" program is **not** freely seller-writable via `update_stock` | OFFICIAL | `SEARCH_INDEXED` |

**Do not import** the Mercado Livre `STANDARD` / `MULTI_ORIGIN_*` model. Shopee's
model is its own: item/model stock, an *optional* warehouse/`location_id`
dimension (unknown for BR), plus Shopee-fulfilment stock that is not
seller-owned. Phase 02 safe default: **one seller-managed stock figure per model
in BR**; treat multi-warehouse as a `DYNAMIC` account/region capability check.

---

## 15. Warehouses

| Concept | Finding | Verification |
|---|---|---|
| Seller pickup/warehouse address | a shop setting; `logistics.get_address` → `address_id` (pickup / return / default) | `SEARCH_INDEXED` |
| `location_id` on stock | exists in some markets; **UNRESOLVED for BR** | `UNVERIFIED` |
| Shopee distribution centers | Shopee BR operates a growing DC network for fulfilment / Envios Shopee | `SEARCH_INDEXED` (press) |
| Fulfilment-center stock write | not a seller `update_stock` target | `SEARCH_INDEXED` |

Anything account- or region-specific here is `DYNAMIC` and must be resolved per
shop, never hardcoded.

---

## 16. Brand Model

**Different from Mercado Livre — do not port ML brand semantics.**

| Aspect | Finding | Tag | Verification |
|---|---|---|---|
| Is brand an attribute? | Yes — and in **Shopee Brasil it is a mandatory attribute for the product** | OFFICIAL | `SEARCH_INDEXED` |
| Brand list API | `get_brand_list` (v2), per category → `brand_id`, `original_brand_name`, `display_brand_name` | OFFICIAL | `SEARCH_INDEXED` |
| "No Brand" | **"Sem marca"** is supported and is the **first option** in the list | OFFICIAL | `SEARCH_INDEXED` |
| Seller-registered brands | seller can submit their own brand or the manufacturer's; **subject to Shopee approval**; auto-reverts listings to "Sem marca" if rejected | OFFICIAL | `SEARCH_INDEXED` |
| Rejection reasons | logo/name mismatch, spelling errors, wrong category, policy breaches | OFFICIAL | `SEARCH_INDEXED` |
| Brand-restricted categories | some categories require brand authorisation (IP) | OFFICIAL | `SEARCH_INDEXED`, specifics `UNVERIFIED` |
| Registration API | a `brand/register_brand`-style endpoint is **not confirmed**; may be seller-center-only | — | `UNVERIFIED` |
| Unknown/generic products | select "Sem marca" | OFFICIAL | `SEARCH_INDEXED` |

**Implication for requirement layering:** on Shopee BR, brand is closer to
`PUBLICATION_REQUIRED` (must be set — even if "Sem marca") than to Mercado
Livre's `CONDITIONAL_REQUIRED`. But the *value* is still evidence-gated: a
genuine unbranded product → "Sem marca"; never invent a brand to satisfy the
field. This is a Shopee-specific requiredness, not a shared-core change.

---

## 17. Listing Lifecycle / Status

**`item_status` (v2)** — consistently corroborated set: **`NORMAL`**,
**`UNLIST`**, **`BANNED`**, **`DELETED`**. A **`REVIEWING`** / under-review state
appears in some item-info responses (possibly a separate `review_status`) —
`UNVERIFIED`. `SELLER_DELETE` vs system delete — `UNVERIFIED`.

| State | Meaning (reconstructed) | Verification |
|---|---|---|
| `NORMAL` | live and for sale | `SEARCH_INDEXED` |
| `UNLIST` | exists but hidden / not for sale (seller toggled via `unlist_item`) | `SEARCH_INDEXED` |
| `BANNED` | removed by Shopee for a policy violation | `SEARCH_INDEXED` |
| `DELETED` | removed (by seller or system) | `SEARCH_INDEXED` |
| `REVIEWING` (?) | pending moderation after create/edit | `UNVERIFIED` |
| draft / incomplete | Seller Center UI has a draft state; whether `add_item` can create a non-live draft via API | `UNVERIFIED` |

**Operations** (v2, names `SEARCH_INDEXED`): `add_item`, `update_item`,
`update_item_sku`, `delete_item`, `unlist_item`, `init_tier_variation`,
`update_tier_variation`, `add_model`, `update_model`, `delete_model`,
`get_model_list`, `update_price`, `update_stock`, `get_item_base_info`,
`get_item_extra_info`, `get_item_list`, `search_item`, `category_recommend`,
`get_category`, `get_attributes`, `get_brand_list`, `get_item_limit`,
`get_dts_limit`, `media_space/upload_image`, `media_space/upload_video` (+
`get_video_upload_result`), `logistics/get_channel_list`, `logistics/get_address`.

State model to build in Phase 02: `NOT_CREATED → (add_item) → [REVIEWING?] →
NORMAL ⇄ UNLIST`, with `BANNED` / `DELETED` as terminal-ish enforcement/removal
states, and edits (`update_*`) possibly re-triggering review. ⚠ verify the whole
transition graph.

---

## 18. API Validation

**No dedicated pre-publication validator was found** — there is no analogue of
Mercado Livre's `POST /items/validate` / dry-run.

Pre-checks that *are* available: `category_recommend`, `get_category`,
`get_attributes` (mandatory flags), `get_brand_list`, `get_item_limit`
(numeric bounds), `get_dts_limit`. Final validation is the **`add_item` /
`update_item` response itself** (success, or an error + optional `warning`).

**Consequence for the readiness model:** `PUBLICATION_STATUS` for Shopee must
lean harder on (a) fetching all the limit/attribute endpoints up front and
checking the assembled payload against them locally, and (b) treating the actual
create call as the authoritative gate. Document the missing-validator fact
explicitly in the Skill.

---

## 19. Compliance

### 19.1 Structure

Shopee BR **"Política de Produtos Proibidos e Restritos"** (help.shopee.com.br
art. 76226; seller-edu art. 3304). Two tiers:

- **Proibidos (prohibited):** cannot be listed at all. Includes anything lacking
  required homologation from **ANVISA, ANATEL, INMETRO, MAPA, ANS** (and
  effectively Exército / Polícia Federal for controlled items); counterfeits;
  many used/hygiene items; fractioned prescription meds, hormones, anaesthetics;
  cosmetic testers/samples/decants; minoxidil; etc.
- **Restritos (restricted):** listable **only with authorisation / conditions**.
  Seller requests release via Seller Center ("como solicitar a liberação para
  venda de produtos restritos", edu art. 12544). Examples: eyewear frames,
  food & beverages, alcohol, dairy, certain electronics; remould tyres (must
  carry "Remold" in the title + INMETRO number in the body).

### 19.2 Model to build (matches the Mercado Livre approach)

```
product / claims / context
        ↓  Shopee policy resolution  (dynamic — NOT a frozen catalogue)
ComplianceFinding { rule, source, classification, scope, evidence, status, affects[], remedy }
        ↓
readiness impact
```

States: `ALLOWED` / `RESTRICTED` (with `AUTHORIZATION_REQUIRED` as a sub-state) /
`PROHIBITED` / `UNRESOLVED`. Use Shopee's own **prohibited vs restricted** split
as the backbone.

**Do not freeze the prohibited catalogue into the Skill** (brief §35). Build the
resolution *procedure* and keep the examples as illustrations tagged
`SEARCH_INDEXED` + `⚠ verify`.

### 19.3 Contact info / external diversion

Prohibited in titles, descriptions, images, shop decoration, and package
inserts: links, phone numbers, QR codes, social handles, WhatsApp invitations,
"pagar fora", any redirection off Shopee. Treated as a **removable string**
(drop → `CONTENT_STATUS` can stay `PASS`; `PUBLICATION_STATUS = FAIL` only if
still in the payload at publish) — identical to the Mercado Livre handling.

---

## 20. Regulatory Requirements (Brazil)

| Regulator | Scope | Shopee handling (reconstructed) | Verification |
|---|---|---|---|
| **ANVISA** | cosmetics, health/food products, supplements, some devices | listing needs valid registration/notification; fractioned Rx meds, hormones, anaesthetics prohibited; testers/samples prohibited | `SEARCH_INDEXED` |
| **ANATEL** | telecom / RF devices (wi-fi, Bluetooth) | homologation required; uncertified devices prohibited | `SEARCH_INDEXED` |
| **INMETRO** | toys, electricals, appliances, auto parts, textiles, tyres, PPE, childcare | certification required; certificate/registration number often required in listing; remould tyres must show INMETRO number | `SEARCH_INDEXED` |
| **MAPA** | agri / veterinary / pet food / some foods | registration required; some items prohibited | `SEARCH_INDEXED` |
| **ANS** | health-plan-related | referenced in the policy as a homologation authority | `SEARCH_INDEXED` |
| Exército / PF | controlled products (airguns, chemicals, etc.) | heavily restricted / prohibited | `SEARCH_INDEXED` |

Principle (shared with Mercado Livre): **possible applicability → resolve**;
**confirmed applicable + missing requirement → `PUBLICATION_STATUS` blocker**.
Never assume applicability; never invent a registration number.

---

## 21. Moderation / Enforcement

- **Post-publication.** Non-compliant listing → hidden / `BANNED`; may also draw
  **penalty points**.
- **Penalty-point system (BR):** points accrue **weekly**, valid **60 days**;
  each infraction has a point value (e.g. undisclosed shipping charge ≈ 10 =
  critical; late-dispatch ≈ 8; unjustified post-payment cancel ≈ 5; **incomplete
  listing ≈ 3**). Thresholds trigger **progressive sanctions**: search
  deprioritisation → feature limits → listing-count limits → account
  suspension. Official Shopee BR PDF "Pontos de Penalidade" (S5).
- **Appeals / correction** flow exists via Seller Center.
- **API exposure:** partial — `item_status = BANNED` is visible; a
  penalty/violations/account-health API for **BR is `UNVERIFIED`** (other Shopee
  markets expose `public/get_shop_penalty`, account-health endpoints — BR
  availability unconfirmed). Much of the enforcement state is **Seller-Center
  only**.
- Keep **pre-publication readiness** separate from **post-publication
  enforcement** (brief §40). Enforcement state ≠ ProductMaster truth. Preserve
  `reason` + `remedy`; never fabricate a remedy; never recreate a listing to
  dodge a ban.

---

## 22. Quality / Seller Signals

| Signal | Level | Exposure | Product-truth impact |
|---|---|---|---|
| Penalty points | shop | Seller Center; API `UNVERIFIED` for BR | none — feeds `EXECUTION` / compliance findings, not `CONTENT` |
| Shop performance (late-shipment rate, non-fulfilment rate, chat response, rating) | shop | Seller Center "Métricas da Loja"; API partial | none — never folded into product truth |
| Preferred / Mall / star tiers | shop | Seller Center | none |
| Listing completeness / quality | listing | Seller Center hints; **no confirmed listing-quality *score* API for BR** (`get_item_limit` = limits, not a grade) | feeds `QUALITY_STATUS` only |

No `/performance`-style per-listing quality endpoint is confirmed for Shopee BR.
`QUALITY_STATUS` will be driven mostly by *our own* completeness/optimisation
checks, plus penalty-point exposure as an `EXECUTION`/compliance input.

---

## 23. Product Identity Guard — is it marketplace-independent?

**Yes — `SHARED_CORE_CANDIDATE` (reinforced by Shopee).**

- Shopee's tightened **image moderation** (2026) and its **return reasons**
  (item not as described, wrong variation, missing parts, size mismatch,
  material/colour mismatch, compatibility) both punish exactly the gap the Guard
  protects: *presentation drifting from product identity*.
- The protected facts are not Shopee-specific: geometry, colour, material
  appearance, finish, components, branding, quantity, included accessories,
  packaging contents, variant, scale.
- The "presentation may change; identity may not" split, and the
  `IDENTITY_PASS / REVIEW / FAIL` audit on every generated/edited asset (a
  generated image is an output, never evidence), transfer unchanged.

Mark `SHARED_CORE_CANDIDATE`; do **not** extract globally yet (only two
marketplaces).

---

## 24. Video

| Surface | Rules found | Scope |
|---|---|---|
| **Product listing video** | Shopee supports one video on the listing (`media_space/upload_video` → `video_upload_id`). Duration/size/aspect for the *listing* video: `UNVERIFIED` for BR. | **listing** |
| **Shopee Video** (short-form social feed) | 60 s max (≤ 30 s preferred), **9:16 vertical**, no borders, one product in focus, originality tiers for creators, community guidelines (seller-edu art. 20631) | **Shopee Video only** |
| **Shopee Live** | its own prohibited-products policy (help art. 188686) and its own content rules | **Shopee Live only** |

**Every video rule carries a `scope` field.** A Shopee Video / Shopee Live rule
must **not** be applied to the product listing unless Shopee explicitly makes it
cross-surface. Same caution for prohibited-content, image, title, link and
regulated-goods rules discovered on the Live/Video surfaces (brief §46).

---

## 25. Proposed Readiness Model

The Mercado Livre four dimensions **fit Shopee**. Recommendation: **keep four,
add none.**

| Dimension | Shopee meaning | FAIL triggers (all require a *confirmed* condition) |
|---|---|---|
| `CONTENT_STATUS` (`PASS`/`REVIEW`/`FAIL`) | Can we truthfully represent the product? Marketplace-independent. | core identity/function unknown; conflicting evidence; a model's identity unestablished; content needs a fabricated fact; content represents a materially different product; `IDENTITY_FAIL` with no truthful option. A removable policy string is **not** a CONTENT failure. |
| `PUBLICATION_STATUS` (`PASS`/`REVIEW`/`FAIL`) | Does the assembled listing meet **resolved** Shopee requirements? | not a leaf category; a mandatory attribute missing; **brand attribute unset** (BR); 0 compliant 1:1 images or an image over/under a confirmed limit; missing weight; no enabled logistics channel; title/description over a confirmed limit; price outside `price_limit`; confirmed prohibited product; confirmed applicable regulated requirement with missing evidence; assembled payload still carrying contact/diversion content at publish |
| `EXECUTION_STATUS` (`PASS`/`REVIEW`/`FAIL`) | Can **the target Open Platform operation** run now for this shop? Evaluated **per operation**. | invalid/absent shop auth, token or scope; rate-limited; `item BANNED` for an update op; model-scoped op with no `model_id`; multi-warehouse write where BR doesn't support it; **and never**: requiring an `item_id`/`model_id` that the create op itself returns |
| `QUALITY_STATUS` (`PASS`/`REVIEW` only — no FAIL) | Complete & optimised beyond bare validity | recommended attributes unfilled; no 3:4 images; thin description; weak SEO title; penalty-point exposure noted → `REVIEW` |

- **Derived compatibility `status`:** `FAIL` if any of CONTENT/PUBLICATION/
  EXECUTION = FAIL; else `REVIEW` if any dimension (incl. QUALITY) = REVIEW; else
  `PASS`. (Same as the post-08A Mercado Livre model.)
- **Pending vs FAIL (brief §50):** not checked → `REVIEW`; checked and
  incompatible → `FAIL`; unknown ≠ FAIL. A non-empty `dynamic_checks_required`
  is never an automatic FAIL.
- **Does Shopee force a 5th dimension?** No. Penalty-point / account-health state
  is post-publication enforcement → routes to `EXECUTION` + a compliance
  finding (like ML moderation). Listing-quality grade → `QUALITY`.

### 25.1 Operation-scoped EXECUTION — Shopee operations

`CREATE_ITEM` (`add_item`), `INIT_VARIATION` (`init_tier_variation`/`add_model`),
`UPDATE_ITEM`, `UPDATE_PRICE`, `UPDATE_STOCK`, `UNLIST_ITEM`, `DELETE_ITEM`,
`UPLOAD_MEDIA`, `FETCH_ITEM`. Each has its own prerequisites; `CREATE_ITEM` must
not require the `item_id` it will return.

---

## 26. Shared-Core Candidates

| Concept | Verdict | Why |
|---|---|---|
| **Evidence Model** (CONFIRMED / INFERRED / MISSING / CONFLICTING / UNSUPPORTED) | `SHARED_CORE_CANDIDATE` | Nothing Shopee changes the need; competitor claim ≠ our fact still holds. |
| **Requirement Layers** (CORE / CONDITIONAL / PUBLICATION / COMMERCIAL) | `SHARED_CORE_CANDIDATE` | Fits; contents differ (brand is PUBLICATION-ish on Shopee BR, GTIN is not global). |
| **Readiness Model** (4 dimensions + derived status + pending≠FAIL) | `SHARED_CORE_CANDIDATE` (strongest) | Transfers with only vocabulary changes. |
| **Product Identity Guard** | `SHARED_CORE_CANDIDATE` | Reinforced by Shopee image moderation + return reasons. |
| **Claim Safety** (`claim strength ≤ evidence strength`) | `SHARED_CORE_CANDIDATE` | Shopee misleading-claims / health-claims policy reinforces it. |
| **Compliance Finding** (resolution procedure, states, `affects[]`) | `SHARED_CORE_CANDIDATE` | Shopee prohibited/restricted maps onto the state set; keep dynamic. |
| **Return Prevention** ("could a reasonable buyer expect a materially different product?") | `SHARED_CORE_CANDIDATE` | Shopee return reasons match; Shopee adds a variation/model-confusion hotspot. |
| **Marketplace Mapping** (internal id ↔ external ids; listing is a projection, never ProductMaster truth) | `SHARED_CORE_CANDIDATE` | `shop_id`/`item_id`/`model_id` ↔ `variant_id`; direction of truth unchanged. |
| **Dynamic Checks** (registry; pending→REVIEW; checked-incompatible→FAIL) | `SHARED_CORE_CANDIDATE` | Shopee needs it *more* (more values are dynamic). |
| **Source Verification Quality** (LIVE / SEARCH_INDEXED / UNVERIFIED) | `SHARED_CORE_CANDIDATE` | Already reused verbatim in this report. |

Every one is also `NEEDS_MORE_MARKETPLACES` for **extraction** — do not lift
anything into a shared core with only Mercado Livre + Shopee. Re-test at
marketplace #3.

**Mercado-Livre-specific (confirmed NOT shared):** User Product, Family,
PARENT_PK / CHILD_PK, Multi Origem (STANDARD / MULTI_ORIGIN_*), `listing_allowed`,
`EMPTY_GTIN_REASON`, `catalog_product_id` / Buy Box, `POST /items/validate`,
`GET /item/$ID/performance`, `moderations/last_moderation`, `domain_discovery`.

---

## 27. Mercado Livre Comparison

| Concept | Mercado Livre | Shopee (Brasil) | Shared candidate? |
|---|---|---|---|
| Listing entity | Item (`MLBxxxx`); sometimes bound to a User Product / Catalog product | **item** (`item_id`) — standalone | abstraction: "listing entity" — yes |
| Variant entity | legacy `variations[]` **or** User Product + sale-condition Items | **model** (`model_id`) = one combination of ≤2 tier options | "sellable variant" — yes |
| Variant identity | `SELLER_SKU` attribute; PARENT_PK/CHILD_PK derive the family | `model_sku` (free text, seller-set, mutable); tiers are positional | yes (stable-id mapping) |
| Category | predictor `domain_discovery/search` → validate leaf + `listing_allowed` | `category_recommend` → validate leaf via `get_category` | yes (discover→resolve→validate) |
| Attributes | `GET /categories/$ID/attributes` + `conditional` POST resolution; `required`/`conditional_required`/`catalog_required` tags | `get_attributes` with `is_mandatory`; **no conditional-resolution endpoint found** | partly — "resolve attributes dynamically" yes; conditional mechanism ML-specific |
| Brand | CONDITIONAL attribute; generic products legitimately have none | **mandatory attribute (BR)**; explicit **"Sem marca"**; seller brand registration + Shopee approval | concept yes; requiredness Shopee-specific |
| GTIN / identifiers | category-conditional; `EMPTY_GTIN_REASON` structured absence | EAN "obrigatório para alguns produtos"; **no structured absence mechanism** | concept yes; ML's `EMPTY_GTIN_REASON` ML-specific |
| Title | manual mode vs generated `family_name`; DYNAMIC `max_title_length`; no "first 40" rule | single `item_name`; ≈255–256 (BR, DYNAMIC); Title Case; no diversion/spam | yes (constraint vs policy vs recommendation split) |
| Description | `plain_text`; complements ficha técnica | ≈5,000 chars; `normal`/`extended`; no links/contact | yes |
| Images | JPG/PNG; min 500×500; DYNAMIC `max_pictures_per_item[_var]`; category cover rules; moderation | 1–9; **1:1 mandatory**, 3:4 recommended; min ≈350×350 (DYNAMIC); cover text ban; ≥60% frame; moderation | yes (3-layer model: OFFICIAL / gallery strategy / Identity Guard) |
| Video | limited; surface-scoped | listing video + **separate** Shopee Video / Shopee Live surfaces, each with own policy | yes — and Shopee makes `scope` on video rules *essential* |
| Price | Item / sale-condition concern; listing types | model or item level; BRL; `price_limit`; range display | yes (commercial context, not product truth) |
| Stock | `available_quantity` in STANDARD; `seller_warehouse` endpoints once Multi Origem active | model/item `update_stock`; `seller_stock` (+ `location_id` in some markets — **UNRESOLVED for BR**) | concept yes; ML STANDARD/MULTI_ORIGIN model **not** shared |
| Warehouses | Multi Origem warehouses = same state as CNPJ; `meli_facility` (Full) not writable | seller pickup `address_id`; Shopee DCs / Envios Shopee not seller-writable; multi-warehouse BR unknown | partial |
| Publication validation | `POST /items/validate` (204 / 400 `cause[]`) | **none found** — the `add_item` response is the gate | ML-specific endpoint; the *need* is shared |
| Compliance | prohibited/restricted/regulated resolution procedure; `ComplianceFinding` | prohibited vs restricted (authorisation) tiers; same regulators family | yes (resolution procedure + finding struct) |
| Moderation | `moderations/last_moderation/{ref}` + `moderations/infractions/{user}`; infraction states | `item_status = BANNED`; **penalty points** (weekly, 60-day); BR violations API `UNVERIFIED` | concept yes; endpoints ML-specific; Shopee adds points system |
| Quality signal | `GET /item/$ID/performance` (replaces `/health`) | **no confirmed listing-quality API (BR)**; our own checks + penalty exposure | `QUALITY_STATUS` dimension yes; endpoint ML-specific |
| External IDs to persist | `item_id`, User Product id, family id, catalog id | `shop_id`, `item_id`, `model_id`, `brand_id`, media ids | yes (mapping discipline) |

---

## 28. Proposed Skill Structure

Start from the brief's candidate, adjusted for what discovery found (fold
warehouses into inventory+logistics; split brand/identifiers out because Shopee
BR makes brand mandatory and GTIN conditional; keep `video.md` because
surface-scoping is a real risk here; add `moderation-and-enforcement.md` because
penalty points are prominent; add `api-and-auth.md` because Open Platform BR
access is an open question).

```
.claude/skills/shopee-listing-best-practices/
├── SKILL.md                       # entry: what/why, requirement layers, tag set,
│                                  # §reference map, "when to call the API",
│                                  # creation workflow, readiness model, output JSON
└── references/
    ├── official-sources.md        # source map + verification quality; every row ⚠ verify
    ├── product-and-variation-model.md   # shop / item / tier_variation / model; combination semantics; id mapping
    ├── categories.md              # get_category, category_recommend; discover→resolve→validate
    ├── attributes.md              # get_attributes; is_mandatory; recommended≠required; regulatory attrs
    ├── brand-and-identifiers.md   # brand mandatory attr + "Sem marca" + registration/approval; EAN/GTIN conditional
    ├── titles-and-seo.md          # length (DYNAMIC), Title Case, policy vs recommendation vs SEO hypothesis
    ├── descriptions.md            # ~5000 chars, normal/extended, no links/contact
    ├── images.md                  # OFFICIAL specs (1:1, 1–9, cover rules, moderation) / gallery strategy / Product Identity Guard
    ├── video.md                   # listing video ONLY; Shopee Video / Shopee Live explicitly scoped OUT
    ├── pricing.md                 # model/item price, price_limit, BRL, misleading-price policy
    ├── inventory-and-logistics.md # model/item stock, seller_stock, warehouse/location (UNRESOLVED BR), channels, weight/dims, days_to_ship, pre_order, condition
    ├── compliance.md              # prohibited vs restricted(authorisation) vs regulated; claims safety; IP/brand; contact-diversion as removable string
    ├── moderation-and-enforcement.md  # item_status graph; penalty points; appeals; API exposure gaps
    ├── quality-audit.md           # 4 readiness dimensions, aggregation, gates, output JSON, dimension scores
    ├── return-prevention.md       # materially-different-product test; Shopee model/variation-confusion hotspot
    ├── competitor-research.md     # analyse without copying; competitor listing ≠ Shopee policy
    ├── review-mining.md           # buyer objections as market evidence, not product-fact evidence
    └── api-and-auth.md            # Open Platform: partner_id/shop_id/token/refresh/sign/scopes/limits/sandbox; **BR access risk**; dynamic-check registry
```

Change it further if Phase 02 discovery (real portal access) contradicts the
model above.

---

## 29. Unverified Facts (consolidated)

| # | Unknown | Why it matters | Safe temporary behavior | How to verify |
|---|---|---|---|---|
| U1 | Full v2 request schema for `add_item` / `add_model` / `init_tier_variation` (field names, requiredness, nesting) | Can't assemble a correct payload or a real `PUBLICATION_STATUS` check | Treat all field names as provisional; every rule `⚠ verify`; rely on the `add_item` error response as the gate | Read `open.shopee.com` v2 product docs (maintainer with portal access) or call the API in sandbox |
| U2 | **Is the Open Platform API available to Brazil partners/shops?** | Entire `EXECUTION` layer; whether a Skill→API pipeline is even possible for BR | Assume manual/Seller-Center publishing until confirmed; keep the Skill output publish-agnostic | Shopee Open Platform onboarding for a BR shop; Shopee partner support |
| U3 | Title max/min length (BR) | `PUBLICATION_STATUS` hard check | Use `get_item_limit`; do not hardcode ≈255 | `get_item_limit` response; live seller-edu page |
| U4 | Image min/max dimensions, file types, byte cap (BR) | `PUBLICATION_STATUS` + moderation | Use `get_item_limit`; treat ≈350×350 / ≈1024 as provisional | `get_item_limit`; live image-rules article |
| U5 | Description limit + `extended` availability (BR) | content assembly | Assume ≈5,000 plain-text; don't rely on `extended` | live docs; API |
| U6 | Variation caps: max tiers (2?), max options/tier, max models | variation planning | Assume 2 tiers; flag anything larger for review | `get_item_limit`; API docs |
| U7 | Stock model for BR: single pool vs multi-warehouse `location_id`; absolute vs incremental `update_stock` | `EXECUTION_STATUS` for `UPDATE_STOCK`; inventory design | Assume single seller pool, absolute set | API `get_item_base_info` / `stock` docs for a BR shop |
| U8 | Whether Shopee BR has any catalogue / "produto" matching / shared-page concept | whether to build association logic at all | Assume every listing standalone | Seller Center "Configuração do Produto"; portal |
| U9 | `REVIEWING` / draft states; full `item_status` transition graph; whether edits re-trigger review | lifecycle + `EXECUTION` gating | Model `NORMAL/UNLIST/BANNED/DELETED`; mark review states unknown | API docs; observed behavior |
| U10 | GTIN/EAN: which categories require it, at item or model level, duplicate rules | `PUBLICATION_STATUS` for regulated/again categories | Treat as `DYNAMIC` per category; never invent a code | `get_attributes` per category; live policy |
| U11 | Brand registration API + brand-authorisation categories | `EXECUTION` for brand submission; `PUBLICATION` for IP-gated categories | "Sem marca" fallback; flag branded IP-sensitive categories for review | portal `brand/*` docs; Seller Center |
| U12 | Penalty/violations/account-health API for BR | `EXECUTION` + compliance findings post-publication | Treat enforcement state as Seller-Center-only; require human input | portal `public/get_shop_penalty` / account-health for BR |
| U13 | Rate limits (per endpoint, per shop) for BR | `EXECUTION` throttling | Assume conservative (~10 rps / ~1000 min); backoff on `error_rate_limit` | portal; response headers |
| U14 | Listing video constraints (duration/size/aspect) vs Shopee Video constraints | `video.md` correctness; surface-scoping | Only state Shopee Video/Live numbers with explicit scope; mark listing-video unknown | portal `media_space/upload_video`; live seller-edu |
| U15 | `condition = USED` allowed categories; refurbished as a value | `PUBLICATION_STATUS` for used goods | Default `NEW`; treat `USED` as category-gated + compliance-checked | `get_category` / live policy |
| U16 | Signature base string exact composition & param ordering; SIP/`merchant_id` flows | `EXECUTION` auth | Use the reconstructed base `partner_id+path+timestamp+access_token+shop_id`; ⚠ verify | portal auth docs; sandbox |

---

## 30. Risks

| Risk | Severity | Mitigation |
|---|---|---|
| **API portal unreadable from tooling** → every API rule is `SEARCH_INDEXED`/`UNVERIFIED` | HIGH | Phase 02 scaffolds with everything `⚠ verify`; a maintainer with `open.shopee.com` access transcribes the v2 product/auth pages; or the MCP layer calls sandbox and records real responses |
| **Open Platform may not be open to BR partners** (U2) | HIGH | Design the Skill's output to be publish-agnostic (draft + audit JSON, like the Mercado Livre Skill); don't assume an execution pipeline; resolve availability before building any client |
| Hardcoding a wrong limit (title ≈255, image ≈350) | MEDIUM | All limits are `DYNAMIC` via `get_item_limit`; the "≈" values live only in prose as provisional, never in a rule |
| Applying Shopee Video / Shopee Live policy to product listings | MEDIUM | `scope` field mandatory on every media/content rule; `video.md` explicitly excludes Live/Video |
| Importing Mercado Livre concepts (User Product, Multi Origem, catalog, `/validate`) by reflex | MEDIUM | §26 "ML-specific" list is explicit; SKILL.md must state the non-mapping up front |
| Treating "Shopee recommends X" as a ranking factor | MEDIUM | §10 separates OFFICIAL recommendation from EXPERIMENTAL ranking hypothesis; no ranking claims without a Shopee statement of the relationship |
| Prohibited-catalogue drift (freezing a list that changes) | MEDIUM | `compliance.md` is a resolution *procedure*; examples tagged `⚠ verify` with consultation dates |
| Brazil regulatory scope creep (assuming ANVISA/INMETRO always apply) | LOW-MEDIUM | "possible applicability → resolve"; only "confirmed applicable + missing → blocker" |
| No pre-publication validator → false confidence in `PUBLICATION_STATUS` | MEDIUM | Skill states the gap; `PUBLICATION_STATUS` is "resolved requirements satisfied", the `add_item` call remains authoritative |
| Third-party integrator docs (S10) contradicting each other on numbers | LOW-MEDIUM | Prefer S2/S3 snippets over S10; record confidence; resolve via API |

---

## 31. Recommended Implementation Sequence (Phase 02 preview)

See §65 for the numbered sequence and the Phase 01 decision.

---

## Fact Table (material findings)

| Fact | Classification | Source | Verification | Confidence |
|---|---|---|---|---|
| The **item** (`item_id`) is the listing; the **model** (`model_id`) is the sellable variant | OFFICIAL | S7, S9, S10 | SEARCH_INDEXED | HIGH |
| A model = one **combination** of ≤2 tier options (`tier_index`), with its own `model_sku`, price, stock | OFFICIAL | S6, S7 | SEARCH_INDEXED | MEDIUM-HIGH |
| `item_sku` / `model_sku` are seller-set, mutable, (largely) buyer-invisible | OFFICIAL | S6, S10 | SEARCH_INDEXED | MEDIUM |
| No User Product / Family / PARENT_PK / Multi Origem / catalog page / `/items/validate` | (absence) | S6–S10 | SEARCH_INDEXED (absence of evidence) | MEDIUM |
| `item_status` ∈ `NORMAL`, `UNLIST`, `BANNED`, `DELETED` | OFFICIAL | S7 | SEARCH_INDEXED | MEDIUM-HIGH |
| Category tree via `get_category`; prediction via `category_recommend`; leaf required | OFFICIAL | S7 | SEARCH_INDEXED | MEDIUM-HIGH |
| Attributes via `get_attributes`; `is_mandatory` per category; no conditional-resolution endpoint | OFFICIAL | S6, S7 | SEARCH_INDEXED | MEDIUM |
| **Brand is a mandatory attribute in Shopee BR**; "Sem marca" is the first option | OFFICIAL | S2, S10 | SEARCH_INDEXED | MEDIUM-HIGH |
| Seller brand registration exists; Shopee-approved; auto-reverts to "Sem marca" on rejection | OFFICIAL | S2, S10 | SEARCH_INDEXED | MEDIUM |
| EAN/GTIN "obrigatório para alguns produtos"; not a blanket listing requirement; no `EMPTY_GTIN_REASON` analogue | OFFICIAL | S10, S11 | SEARCH_INDEXED | MEDIUM |
| Title max ≈ 255–256 chars (BR) | DYNAMIC | S10 | SEARCH_INDEXED | MEDIUM |
| Title: Title Case, no special chars, no diversion/spam/promo terms; shape recommendation only | OFFICIAL | S2, S10 | SEARCH_INDEXED | MEDIUM-HIGH |
| Description ≈ 5,000 chars; no links/phones/QR/social/WhatsApp; `normal`/`extended` types | OFFICIAL | S10 | SEARCH_INDEXED | MEDIUM |
| Images: **1–9**; **1:1 mandatory**; 3:4 recommended; min ≈ 350×350; recommended ≈ 1024×1024 | OFFICIAL / DYNAMIC | S2, S10 | SEARCH_INDEXED | MEDIUM (count HIGH; dims LOW) |
| Cover photo: no commercial text; no watermark/border/logo; product ≥ 60% of frame if used as logo | OFFICIAL | S2 | SEARCH_INDEXED | MEDIUM-HIGH |
| 2026 image-rule tightening in BR; non-compliant images → ban/hide | OFFICIAL | S10 | SEARCH_INDEXED | MEDIUM |
| Price & stock at model level (else item level); BRL; `update_price`/`update_stock` (+ batch) | OFFICIAL | S6, S7 | SEARCH_INDEXED | MEDIUM-HIGH |
| Numeric limits (title/image/price/stock) come from `get_item_limit`; DTS from `get_dts_limit`/category | OFFICIAL | S7 | SEARCH_INDEXED | MEDIUM |
| Stock: `seller_stock`/`stock_info_v2`; multi-warehouse `location_id` in some markets — **UNRESOLVED for BR** | OFFICIAL (other markets) | S7 | UNVERIFIED (BR) | LOW |
| Weight required to publish; package dimensions per channel/category; `days_to_ship` / `pre_order` (7–30 for pre-order) | OFFICIAL | S6 | SEARCH_INDEXED | MEDIUM |
| `condition` ∈ `NEW` / `USED`; USED category-gated in BR | OFFICIAL | S6, S3 | SEARCH_INDEXED | LOW-MEDIUM |
| No dedicated pre-publication validation endpoint | (absence) | S6–S9 | SEARCH_INDEXED (absence) | MEDIUM |
| Shopee BR Prohibited/Restricted policy; regulators ANVISA/ANATEL/INMETRO/MAPA/ANS; homologation-absent → prohibited | OFFICIAL | S3 | SEARCH_INDEXED | MEDIUM-HIGH |
| Restricted items sellable only after Seller-Center authorisation request | OFFICIAL | S2 | SEARCH_INDEXED | MEDIUM |
| Contact info / external diversion prohibited in titles, descriptions, images, inserts | OFFICIAL | S3, S10 | SEARCH_INDEXED | HIGH |
| Penalty-point system (BR): weekly accrual, 60-day validity, progressive sanctions; incomplete listing ≈ 3 pts | OFFICIAL | S5, S10 | SEARCH_INDEXED | MEDIUM |
| No confirmed listing-quality *score* API for BR (no `/performance` analogue) | (absence) | S7 | SEARCH_INDEXED (absence) | LOW-MEDIUM |
| Auth: `partner_id`/`partner_key`; OAuth → `access_token` (4 h) + `refresh_token` (30 d); HMAC-SHA256 sign; timestamp in seconds; host `partner.shopeemobile.com` | OFFICIAL | S8, S9 | SEARCH_INDEXED | MEDIUM-HIGH |
| One app authorises many shops; token/`shop_id` stored per shop; `merchant_id` for SIP/CB | OFFICIAL | S8 | SEARCH_INDEXED | MEDIUM |
| Whether Open Platform API access is available to BR partners/shops | — | — | UNVERIFIED | LOW |
| Shopee Video (60 s, 9:16) and Shopee Live have **separate** policies from product listings | OFFICIAL | S3, S4 | SEARCH_INDEXED | MEDIUM-HIGH |

---

## API Table (endpoints — all v2 unless noted; paths `SEARCH_INDEXED`, schemas `UNVERIFIED`)

| API / Resource | Method | Purpose | Required context | Verification |
|---|---|---|---|---|
| `/api/v2/shop/auth_partner` | redirect | seller authorises the app for a shop → auth `code` | `partner_id`, `sign`, redirect URL | SEARCH_INDEXED |
| `/api/v2/auth/token/get` | POST | exchange `code` → `access_token` + `refresh_token` | `partner_id`, `code`, `shop_id`/`main_account_id` | SEARCH_INDEXED |
| `/api/v2/auth/access_token/get` | POST | refresh `access_token` (rotates refresh token) | `partner_id`, `refresh_token`, `shop_id` | SEARCH_INDEXED |
| `/api/v2/product/category_recommend` | GET | predict category from item name | token, `shop_id`, `item_name` | SEARCH_INDEXED |
| `/api/v2/product/get_category` | GET | category tree (BR) | token, `shop_id`, `language` | SEARCH_INDEXED |
| `/api/v2/product/get_attributes` | GET | attributes for a category (`is_mandatory`, `input_type`, value list) | token, `shop_id`, `category_id`, `language` | SEARCH_INDEXED |
| `/api/v2/product/get_brand_list` | GET | brands for a category (`brand_id`, names) | token, `shop_id`, `category_id`, paging | SEARCH_INDEXED |
| `/api/v2/product/get_item_limit` | GET | numeric limits: name length, image count/size, price/stock bounds, DTS | token, `shop_id`, `category_id` | SEARCH_INDEXED (response shape UNVERIFIED) |
| `/api/v2/product/get_dts_limit` | GET | days-to-ship min/max for a category | token, `shop_id`, `category_id` | SEARCH_INDEXED |
| `/api/v2/product/add_item` | POST | create a listing | token, `shop_id`, full item payload | SEARCH_INDEXED (schema UNVERIFIED) |
| `/api/v2/product/update_item` | POST | edit listing fields | token, `shop_id`, `item_id`, changed fields | SEARCH_INDEXED |
| `/api/v2/product/update_item_sku` | POST | set `item_sku` | token, `shop_id`, `item_id`, `item_sku` | SEARCH_INDEXED |
| `/api/v2/product/delete_item` | POST | delete a listing | token, `shop_id`, `item_id` | SEARCH_INDEXED |
| `/api/v2/product/unlist_item` | POST | list/unlist toggle (batch) | token, `shop_id`, `item_id` + `unlist` bool | SEARCH_INDEXED |
| `/api/v2/product/init_tier_variation` | POST | set the tier structure + initial models on a no-variation item | token, `shop_id`, `item_id`, `tier_variation`, `model` list | SEARCH_INDEXED |
| `/api/v2/product/update_tier_variation` | POST | edit tier option names/images | token, `shop_id`, `item_id`, `tier_variation` | SEARCH_INDEXED |
| `/api/v2/product/add_model` | POST | add models (combinations) | token, `shop_id`, `item_id`, `model_list` | SEARCH_INDEXED |
| `/api/v2/product/update_model` | POST | edit a model (price/stock/sku) | token, `shop_id`, `item_id`, `model_id`, fields | SEARCH_INDEXED |
| `/api/v2/product/delete_model` | POST | remove a model | token, `shop_id`, `item_id`, `model_id` | SEARCH_INDEXED |
| `/api/v2/product/get_model_list` | GET | list models + tier structure of an item | token, `shop_id`, `item_id` | SEARCH_INDEXED |
| `/api/v2/product/update_price` | POST | price (item or via model list); batch variant exists | token, `shop_id`, `item_id`, price list | SEARCH_INDEXED |
| `/api/v2/product/update_stock` | POST | stock (item or model); batch variant exists | token, `shop_id`, `item_id`, stock list | SEARCH_INDEXED |
| `/api/v2/product/get_item_base_info` | GET | core listing fields | token, `shop_id`, `item_id_list` | SEARCH_INDEXED |
| `/api/v2/product/get_item_extra_info` | GET | sales, views, likes, etc. | token, `shop_id`, `item_id_list` | SEARCH_INDEXED |
| `/api/v2/product/get_item_list` | GET | listing ids by `item_status` (`NORMAL`/`UNLIST`/`BANNED`/`DELETED`) | token, `shop_id`, `offset`, `page_size`, `item_status` | SEARCH_INDEXED |
| `/api/v2/product/search_item` | GET | search shop listings | token, `shop_id`, query | SEARCH_INDEXED |
| `/api/v2/media_space/upload_image` | POST | upload image → `image_id` (URLs expire — store the id) | token (image upload may be partner-level), file | SEARCH_INDEXED |
| `/api/v2/media_space/upload_video` / `get_video_upload_result` | POST/GET | listing video upload → `video_upload_id` | token, file | SEARCH_INDEXED (constraints UNVERIFIED) |
| `/api/v2/logistics/get_channel_list` | GET | enabled logistics channels for the shop | token, `shop_id` | SEARCH_INDEXED |
| `/api/v2/logistics/get_address` | GET | pickup / return / default `address_id` | token, `shop_id` | SEARCH_INDEXED |
| `public/get_shop_penalty` / account-health | GET | penalty points / shop health | token, `shop_id` | **UNVERIFIED for BR** |
| pre-publication validate / dry-run | — | — | — | **NOT FOUND** (no analogue of ML `/items/validate`) |

---

## Entity Table

| Entity | Meaning | Internal / external | Stable? | Assigned by | Relationship | API identifier |
|---|---|---|---|---|---|---|
| ProductMaster | marketplace-independent product truth | internal | stable | Product Factory | root | — |
| internal variant / SKU | canonical sellable unit in Product Factory | internal | stable | Product Factory | 1 ProductMaster → N variants | internal `variant_id` |
| Shopee **shop** | seller storefront on Shopee BR | external | stable | Shopee | 1 marketplace account → 1..N shops | `shop_id` |
| Shopee **item** | the listing | external | stable (life of listing) | Shopee (on `add_item`) | 1 shop → N items; 1 ProductMaster → 1 item (per shop) | `item_id` (aka `product_id` in some v2 docs) |
| **tier_variation** | the item's variation axes (≤ 2 tiers) | external, positional, no id | mutable pre-sales; restricted after | seller-defined | 1 item → 0..2 tiers | positional index |
| Shopee **model** | the sellable variant = one combination of tier options | external | stable (life of model) | Shopee (on `add_model` / `init_tier_variation`) | 1 item → N models; 1 internal variant → 1 model | `model_id` + `tier_index[]` |
| `item_sku` / `model_sku` | seller-controlled identity string | seller-set (external field) | mutable | seller / Product Factory | store internal SKU here | value only |
| Shopee **brand** | brand attribute value | external (or seller-registered → approved) | stable | Shopee / seller+approval | 1 item → 1 brand (mandatory BR) | `brand_id` |
| category | leaf category | external | stable-ish (tree changes) | Shopee | 1 item → 1 leaf | `category_id` |
| attribute / value | category attribute + chosen value | external | stable-ish | Shopee | 1 item → N attributes | `attribute_id`, `value_id` |
| logistics channel | shipping option enabled on the listing | external | stable-ish | Shopee | 1 item → N channels | `logistics_channel_id` |
| media | image / video asset | external | stable; **URL expires** | Shopee | 1 item → 1..9 images (+ tier-1 option images) + 1 video | `image_id`, `video_upload_id` |
| `merchant_id` | groups shops for SIP / cross-border | external | stable | Shopee | 1 merchant → N shops | `merchant_id` |

---

## Dynamic Check Table

`when`: which workflow step forces the check. `pending`: behavior if not yet run
(always → `REVIEW`, never `FAIL`). `confirmed incompatibility`: behavior if run
and it proves a mandatory conflict.

| Check | Source / API | When | Pending | Confirmed incompatibility | Affects |
|---|---|---|---|---|---|
| Leaf category valid for the product | `category_recommend` + `get_category` | category step | REVIEW | FAIL | PUBLICATION |
| Mandatory attributes for the category | `get_attributes` (`is_mandatory`) | attributes step | REVIEW | FAIL (missing mandatory) | PUBLICATION |
| Recommended attributes present | `get_attributes` (recommended set) | quality step | REVIEW | — (never FAIL) | QUALITY |
| Brand value resolvable ("Sem marca" or a real, evidence-backed brand) | `get_brand_list` | brand step | REVIEW | FAIL if brand attr unset (BR) | PUBLICATION |
| Brand-authorisation categories | `get_category` / policy | compliance step | REVIEW | FAIL if IP-gated + no authorisation | PUBLICATION / EXECUTION |
| Custom brand approval status | Seller Center / `brand/*` (UNVERIFIED) | brand step | REVIEW | — (fallback "Sem marca") | EXECUTION / QUALITY |
| Title length within limit | `get_item_limit` | title step | REVIEW | FAIL (over limit) | PUBLICATION |
| Image count / dimensions / ratio / bytes | `get_item_limit` + moderation rules | image step | REVIEW | FAIL (0 compliant 1:1, or over/under a hard bound) | PUBLICATION |
| Description length within limit | `get_item_limit` / live docs | description step | REVIEW | FAIL (over limit) | PUBLICATION |
| Variation caps (tiers / options / models) | `get_item_limit` | variation step | REVIEW | FAIL (exceeds cap) | PUBLICATION |
| Price within `price_limit` | `get_item_limit` | pricing step | REVIEW | FAIL (out of bounds) | PUBLICATION |
| Stock model: single pool vs multi-warehouse (`location_id`) for BR | API `get_item_base_info` / stock docs | inventory step | REVIEW | FAIL if op writes a stock shape BR rejects | EXECUTION |
| `update_stock` absolute vs incremental | API docs | UPDATE_STOCK op | REVIEW | FAIL if semantics assumed wrong (guarded) | EXECUTION |
| Days-to-ship within category limits | `get_dts_limit` / `get_category` | logistics step | REVIEW | FAIL (out of range) | PUBLICATION |
| At least one enabled logistics channel; weight/dims present | `logistics/get_channel_list` | logistics step | REVIEW | FAIL (none enabled / no weight) | PUBLICATION |
| `condition = USED` allowed for the category | `get_category` / policy | classification step | REVIEW | FAIL (USED not allowed) | PUBLICATION |
| GTIN/EAN requiredness for the category | `get_attributes` / policy | attributes step | REVIEW | FAIL if required + absent (no invented code) | PUBLICATION |
| Prohibited-product status | Shopee BR policy resolution | compliance step | REVIEW | FAIL (prohibited) | PUBLICATION / EXECUTION |
| Restricted-product authorisation | Seller Center release flow | compliance step | REVIEW | FAIL (restricted + no authorisation) | PUBLICATION |
| Regulated (ANVISA/ANATEL/INMETRO/MAPA/ANS) applicability + evidence | policy + `get_attributes` | compliance step | REVIEW | FAIL (confirmed applicable + missing) | PUBLICATION |
| Contact / external-diversion strings removed from payload | assembled-payload scan | pre-output | REVIEW | FAIL if still present at publish | PUBLICATION (CONTENT stays PASS if dropped) |
| Shop auth + token validity + scope for the target op | token store / `auth/*` | every EXECUTION op | REVIEW | FAIL (no valid token/scope) | EXECUTION |
| Rate-limit headroom | response headers / `error_rate_limit` | every EXECUTION op | REVIEW | FAIL/backoff | EXECUTION |
| Item not `BANNED`/`DELETED` for an update op | `get_item_base_info` | UPDATE_* ops | REVIEW | FAIL (banned/deleted) | EXECUTION |
| Model exists for a model-scoped op | `get_model_list` | UPDATE_PRICE/STOCK/MODEL | REVIEW | FAIL (no model) — but CREATE never needs a not-yet-created id | EXECUTION |
| Penalty-point / account-health state (BR) | `public/get_shop_penalty` (UNVERIFIED) | post-publication | REVIEW | route to compliance finding / EXECUTION | EXECUTION |
| Open Platform API availability for the BR shop | onboarding / partner support | before any EXECUTION design | REVIEW | FAIL the whole execution path (fall back to manual publish) | EXECUTION |

The same four dimensions (`CONTENT` / `PUBLICATION` / `EXECUTION` / `QUALITY`)
absorb every check above — **no fifth dimension is needed.**

---

## 55–65. Report sections mapped

- **55 Discovery Report** — this document (§1–§31 above cover the 31 required
  subsections: Executive Summary, Source Quality, Product/Listing Model, Category,
  Attribute, Variation/Model, SKU & External IDs, Title, Search/SEO, Description,
  Images, Video, Pricing, Inventory, Warehouses, Logistics [§14–15 + §28
  `inventory-and-logistics`], Listing Lifecycle, API Validation, Compliance,
  Regulatory, Moderation/Enforcement, Quality/Seller Signals, Return Prevention
  [§23 + below], Dynamic Checks [table], Proposed Readiness Model, Shared-Core
  Candidates, Mercado Livre Comparison, Proposed Skill Structure, Unverified
  Facts, Risks, Recommended Implementation Sequence [§65]).
- **56 Fact Table** — above.
- **57 API Table** — above.
- **58 Entity Table** — above.
- **59 Dynamic Check Table** — above; the four dimensions are sufficient.
- **60 Uncertainty** — §29 (each row: what is unknown, why it matters, safe
  temporary behavior, how to verify).
- **61 Anti-hallucination** — honored: no endpoint, field, limit, count or rule
  in this report is stated as fact without a `SEARCH_INDEXED`/`UNVERIFIED` mark;
  nothing is inferred from another Shopee country as if it were Brazil (SEA
  values are explicitly flagged); nothing is inferred from Mercado Livre.
- **62 Country scope** — Brazil primary; other markets `REFERENCE ONLY` and
  labelled (multi-warehouse `location_id`, ~20-char option name, ~2 MB image cap,
  `public/get_shop_penalty` are all flagged as other-market / unverified for BR).
- **63 Current date** — 2026-08-28; policy-sensitive findings dated.

### Return Prevention (brief §42 — detail)

The Product Factory test — *"could a reasonable buyer interpret this listing as a
materially different product from what they will receive?"* — transfers to Shopee
unchanged (`SHARED_CORE_CANDIDATE`). Shopee-specific mismatch hotspots to add to
the checklist: **model/variation confusion** (ambiguous option names or model
images → buyer selects the wrong model), quantity/kit contents, size &
measurement, compatibility (fitment), material/colour rendering under Shopee's
image compression, "what's in the box". Shopee return reasons corroborate each.

### Competitor Research (§43) / Review Mining (§44)

Guardrails identical to the Mercado Livre Skill: competitor listing ≠ proof of
Shopee policy; competitor claim ≠ our product fact; reviews = market evidence,
not product-fact evidence unless independently confirmed. Safe uses: vocabulary,
buyer objections, category conventions, image conventions, commercial
positioning. `SHARED_CORE_CANDIDATE` (evidence discipline).

### Shopee Ads (§45)

Kept distinct from organic listing rules. Ads education (S4) may inform
listing-quality *recommendations* only. "Ads docs recommend informative titles /
complete attributes" does **not** become a mandatory organic rule or a confirmed
ranking factor.

---

## 64. No implementation

Nothing in this phase builds an MCP server, an API client, an auth flow, queues,
migrations, agents, a publication path, stock sync, image generation, or a
compliance engine. This report is the knowledge architecture that must precede
them.

---

## 65. Phase 01 Decision

### DISCOVERY GAPS MUST BE RESOLVED FIRST

**Why not "ready":**

1. **No readable API contract.** `open.shopee.com` is unreachable from tooling
   and not usefully indexed. Every v2 endpoint name is corroborated only through
   third-party SDKs/guides; every request schema and every numeric limit is
   `UNVERIFIED`. Writing `PUBLICATION_STATUS` hard checks now would bake in
   guesses.
2. **Execution feasibility is unknown (U2).** Whether the Open Platform API is
   even available to Brazil-domiciled partners/shops is unresolved. The entire
   `EXECUTION` dimension — and any future publish pipeline — hinges on it.
3. **Core limits are low-confidence.** Title length, image dimensions,
   description limit, variation caps, price/stock bounds only exist as `≈`
   `SEARCH_INDEXED` values.
4. **BR inventory/warehouse model unresolved (U7).** Single pool vs multi-
   warehouse `location_id`; absolute vs incremental stock writes.
5. **No pre-publication validator.** The readiness model must be designed around
   its absence, not around an assumed `/validate`.
6. **Catalogue/"produto" concept unresolved (U8).**

None of these block *scaffolding* the Skill (every rule starts `⚠ verify`), but
they block **locking** any rule or building any execution client.

### Proposed — Shopee Skill — Phase 02 (numbered, Mercado-Livre-grade rigor)

- **02.1 — Scaffold.** Create `.claude/skills/shopee-listing-best-practices/`
  with `SKILL.md` + the §28 reference files. Everything tagged; every OFFICIAL
  rule `⚠ verify`; `official-sources.md` records the `SEARCH_INDEXED` posture.
  Port the four-dimension readiness model, evidence model, requirement layers,
  Product Identity Guard, claim-safety, compliance-as-procedure, return-
  prevention test and id-mapping discipline **as Shopee-worded rules**, not as
  copied ML text. State the ML non-mappings up front.
- **02.2 — Resolve the API contract.** A maintainer with `open.shopee.com`
  access (or a sandbox partner account) transcribes the v2 auth + product +
  model + category + attribute + brand + logistics + media pages. Fill the API
  Table schemas; remove `⚠ verify` per row as confirmed.
- **02.3 — Resolve BR access & identity (U2, U16).** Confirm Open Platform
  availability for a BR shop; document partner onboarding, scopes, review,
  sandbox, rate limits, `merchant_id`/SIP. If unavailable → Skill output stays
  publish-agnostic (draft + audit JSON) and `api-and-auth.md` says so.
- **02.4 — Lock the numeric limits (U3–U6, U10).** From `get_item_limit` /
  `get_dts_limit` / `get_attributes` responses for representative BR categories.
  Confirm they are `DYNAMIC` (no constants in the Skill).
- **02.5 — Resolve inventory/warehouse (U7) and lifecycle (U9).** Single vs
  multi-warehouse for BR; stock write semantics; `item_status` transition graph;
  whether edits re-trigger review; draft-via-API.
- **02.6 — Build the Dynamic Check Registry** against real endpoint responses
  (formalize the Dynamic Check Table).
- **02.7 — Compliance procedure** in `compliance.md`: prohibited vs restricted
  vs regulated resolution; the Seller-Center authorisation flow; claims safety;
  IP/brand; contact-diversion as removable string. Examples dated, `⚠ verify`,
  never frozen.
- **02.8 — Moderation & enforcement** in `moderation-and-enforcement.md`:
  `item_status`, penalty points, appeals, API-exposure gaps (U12); enforcement
  state kept separate from ProductMaster truth.
- **02.9 — Quality audit** in `quality-audit.md`: dimensions, sub-scores,
  aggregation, gates, output JSON; mirror the contract into repo `CLAUDE.md` (as
  the ML "Audit / output contract" paragraph does) — as a *second marketplace*,
  not a rewrite of the ML one.
- **02.10 — Catalogue resolution (U8):** confirm whether any matching/shared-page
  concept exists; add `catalog.md` only if it does.
- **02.11 — Adversarial pass.** Scenario tests (prohibited product, missing
  brand, wrong model image, over-limit title, USED in a new-only category,
  regulated item missing INMETRO number, banned-listing update, create-before-id,
  multi-warehouse write on a single-pool shop, contact string in description).
- **02.12 — Cross-marketplace review.** With Mercado Livre + Shopee both stable,
  re-run §26: which `SHARED_CORE_CANDIDATE`s are ready to extract vs still
  `NEEDS_MORE_MARKETPLACES`. **Extraction is a later phase, not 02.**

---

## 67. Guiding principle (restated)

The objective is not to make Shopee fit the Mercado Livre Skill. It is to learn
Shopee accurately enough that, once several marketplace Skills exist, Product
Factory can tell which concepts genuinely belong to its shared core. **Research
first. Abstract later.**

---

## Sources

All consulted 2026-08-28. No source was read `LIVE`; all are `SEARCH_INDEXED`
(search snippets, third-party integrator documentation, or community SDK source)
or `UNVERIFIED`. Summarized, not excerpted.

- Shopee Open Platform API portal — `https://open.shopee.com` — Developers — **UNREACHABLE from tooling** — consulted 2026-08-28 — intended source for every v2 endpoint/schema; nothing read.
- Centro de Educação do Vendedor Shopee BR — `https://seller.shopee.com.br/edu` (arts. 17369 images-3:4, 3304 prohibited/restricted, 12544 restricted-item release, 10619 brand, 20631 Shopee Video guidelines) — Central de Educação — SEARCH_INDEXED (client-rendered; body via snippets) — image ratio/count/cover rules, brand mandatory + "Sem marca", restricted-item authorisation, Shopee Video scope.
- Central de Ajuda Shopee BR — `https://help.shopee.com.br` (art. 76226 Política de Produtos Proibidos e Restritos; art. 188686 Shopee Live prohibited products) — Central de Ajuda — SEARCH_INDEXED — prohibited vs restricted tiers; regulators ANVISA/ANATEL/INMETRO/MAPA/ANS; homologation-absent → prohibited.
- Shopee Ads BR education — `https://ads.shopee.com.br/learn` — Shopee Ads — SEARCH_INDEXED — listing-quality recommendations; video duration; explicitly non-binding for organic.
- "Pontos de Penalidade" (official Shopee BR PDF) — `https://deo.shopeemobile.com/shopee/seller/seller_cms/...Pontos%20de%20Penalidade.pdf` — Shopee BR — SEARCH_INDEXED (referenced, not parsed) — penalty-point mechanics.
- `github.com/wjp-letgo/shopeego` (v2 product package, via pkg.go.dev) — community SDK — SEARCH_INDEXED — v2 endpoint names; `item_status` enum.
- `github.com/teacat/shopeego` (v1, via pkg.go.dev) — community SDK — SEARCH_INDEXED — field names (v1; limits stale); `TierIndex` combination semantics; `condition` / `days_to_ship` / pre-order 7–30.
- `github.com/raviMukti/shopee-api-client`, `github.com/mu-hanz/shoapi` — community SDKs — SEARCH_INDEXED — v2 host URLs; token 4 h / refresh 30 d.
- `developer.inlinex.com.sg/blog/shopee-api-integration-guide-sellers` — third-party — SEARCH_INDEXED — v2 paths (`auth/token/get`, `product/add_item`, `product/get_item_list`, `media_space/upload_image`); sign base string; timestamp in seconds; "store image ids not URLs".
- `rollout.com`, `api2cart.com` (Shopee API guides + 2026 documentation article) — third-party — SEARCH_INDEXED — Open Platform vs Seller API framing; `get_item_base_info`/`update_item`; auth model; rate-limit existence.
- `base.com`, `suporte.anymarket.com.br`, `atendimento.ideris.com.br`, `ajuda.maino.com.br` — BR integrator help centers — SEARCH_INDEXED — title ≈255–256; description ≈5,000; image min ≈350×350 / rec ≈1024; EAN "obrigatório para alguns produtos"; brand mandatory; publish-error example `product.error_busi item_status_invalid`.
- `blog.gs1br.org/como-vender-na-shopee` — GS1 Brasil — SEARCH_INDEXED — EAN/GTIN as best practice, not a blanket Shopee listing requirement.
- `gobots.ai`, `destraveescale.com.br`, `cupomparalelo.com.br`, `1001clicks.com.br`, `mambadigital.com.br`, `giacaglia.com.br` — BR educators/press — SEARCH_INDEXED — penalty-point validity (60 d) & thresholds; brand registration/approval + auto-revert to "Sem marca"; 2026 image-rule tightening; chat/description contact-diversion enforcement.
- Mercado Livre Skill (`.claude/skills/mercado-livre-listing-best-practices/`) — internal — read directly — the architectural reference this report compares against (§26, §27); no rule copied.
