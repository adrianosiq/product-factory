---
name: shopee-listing-best-practices
description: >-
  Create, review and optimize Shopee Brasil listings from a structured
  ProductMaster. Turns product data into a technically plausible, complete,
  honest and competitive listing draft that respects Shopee's current object
  model (Shop, Item, Tier Variation, Model) and minimizes the gap between listing
  expectation and delivered product. Produces a listing draft plus a structured
  quality audit. Never publishes.
last_reviewed: 2026-08-28
review_owner: product-create team
status: PROVISIONAL SCAFFOLD — Phase 02.1
scaffold_notice: >-
  This Skill was scaffolded from Phase 01 discovery, which had ZERO live Shopee
  sources. No readable primary Shopee Open Platform API contract has been
  verified. Every marketplace rule here is provisional and carries a verification
  quality (LIVE / SEARCH_INDEXED / UNVERIFIED); most Shopee-specific rules are
  SEARCH_INDEXED or UNVERIFIED and are marked `⚠ verify`. Numeric limits are NOT
  locked. Do not treat this Skill as a finished marketplace integration. See §13
  "Known discovery gaps".
---

# Shopee (Brasil) — Listing Best Practices — PROVISIONAL SCAFFOLD

## 0. Read this first — what is and is not known

This Skill exists so an agent can reason about a Shopee Brasil listing **without
pretending the marketplace research is finished.** It must keep three states
distinct at all times:

| State | Meaning | How it is marked |
|---|---|---|
| **KNOWN** | Confirmed from a primary Shopee source read directly (`LIVE`), or a Product Factory-internal design decision. | rule tag + `verification: LIVE`; no `⚠ verify` |
| **PROVISIONAL** | Reconstructed from search snippets, third-party integrators or community SDKs (`SEARCH_INDEXED`). Plausible, not confirmed. | rule tag + `verification: SEARCH_INDEXED` + `⚠ verify` |
| **UNVERIFIED** | Not corroborated at all — a schema, limit or behaviour we have only guessed at, or an open question. | `UNVERIFIED` tag + `⚠ verify`; usually also a `dynamic_checks_required` entry |

**As of `last_reviewed` there are no `LIVE` Shopee sources.** `open.shopee.com`
(the Open Platform API portal) was unreachable during discovery; the Shopee BR
seller-education and help centres render client-side and returned only page
titles. Therefore:

- Every Shopee endpoint name, request field, payload shape and numeric limit is
  **`SEARCH_INDEXED` at best, `UNVERIFIED` for all detail.**
- A discovery hypothesis does **not** become `OFFICIAL` because it looked
  plausible in Phase 01. Community SDKs and third-party integration guides are
  **not** primary API sources and never upgrade a rule to `OFFICIAL`.
- Do **not** copy Mercado Livre rules into Shopee (§6.4 lists the concepts that
  are explicitly *not* imported).

## 1. What this Skill is for

Use this Skill when an agent must transform a structured `ProductMaster` into a
**Shopee Brasil listing draft**, or audit an existing draft.

> The goal is the clearest, most complete, most competitive and most faithful
> listing possible for the real product, respecting Shopee's current structure
> and reducing the distance between the expectation the listing creates and the
> product the buyer receives.

When listing appeal and product faithfulness conflict, **faithfulness wins.**

This Skill **never publishes**. It stops at `READY FOR REVIEW` and hands a draft +
audit to a human or a separate pipeline. Whether a Skill→API publish pipeline is
even possible for Shopee Brasil is an **open question** (§13, gap G2) — the output
is deliberately publish-agnostic.

## 2. Product Factory relationship

```
ProductMaster                     ← marketplace-independent product truth
      │  (Shopee adaptation)
      ▼
Shopee listing draft (Item + Tier Variation + Models)
      │  (later, out of scope here)
      ▼
Shopee listing
```

- Direction of truth is **one-way**. A Shopee listing, a competitor's Shopee
  listing, and a Shopee buyer review are **never** sources of `ProductMaster`
  truth.
- Shopee is a **marketplace projection** of the product. Shopee-assigned ids
  (`shop_id`, `item_id`, `model_id`, `brand_id`, media ids) are **external
  mappings** Product Factory persists against its own stable `variant_id` / SKU —
  they never replace internal identity. See `references/product-structure.md`.
- `SHARED_CORE_CANDIDATE`: this mapping discipline looks marketplace-independent,
  but with only two marketplaces implemented it is **not** extracted into a
  shared core. Re-test at marketplace #3.

## 3. Required inputs — requirement layers

Sort every `ProductMaster` field into one layer. The layer decides whether a gap
blocks content, needs review, or is only a warning. **Do not** treat conditional
or marketplace fields as universally mandatory — that creates false blockers.

| Layer | Meaning | A gap here → |
|---|---|---|
| **CORE_REQUIRED** | Needed to identify and truthfully represent the real product: what it physically is / does; the facts that distinguish this item; condition when not obvious; for variants — the variation axes **and each variant's value on them** + a stable per-variant id; evidence for every factual claim. | **BLOCKER** for content creation (`CONTENT_STATUS = FAIL`). |
| **CONDITIONAL_REQUIRED** | Required only when the context calls for it — resolved dynamically (category attributes, compatibility, logistics, regulated status). | Blocks only when a check **confirms** it mandatory and it is missing; otherwise `REVIEW`. |
| **PUBLICATION_REQUIRED** | Not needed to draft; needed before publishing — price, currency, stock, handling time, logistics channels, weight/dimensions, shop/API context, and any Shopee field the create call requires. | `REVIEW` at most on the draft; gated hard only at the separate publish step. |
| **COMMERCIAL_OPTIONAL** | Useful for pricing/strategy, never required for content — acquisition cost, target margin, competitor price targets. | **WARNING**; marks the matching analysis unavailable. Never a blocker. |

**Brand is not automatically CORE_REQUIRED.** Phase 01 evidence indicates brand
is a *material* Shopee Brasil listing attribute and may be required in broad
contexts, but the exact category/API requiredness is `⚠ verify`. Brand belongs to
**publication / category resolution**, not to product identity — unless the
product's identity genuinely depends on brand. A genuine unbranded product uses
**"Sem marca"** (itself `SEARCH_INDEXED`, `⚠ verify`); never invent a brand.
See `references/brand-and-identifiers.md`.

**GTIN / EAN / UPC / ISBN: never invented.** Phase 01 indicates an identifier
*appears* mandatory for some products/categories; exact rules are `⚠ verify`.
Treat as `CONDITIONAL_REQUIRED`, resolved per category. Shopee has **no**
structured "legitimately has no barcode" mechanism that we have confirmed — do
not build an `EMPTY_GTIN_REASON` analogue.

**Recording gaps:** every gap gets a `missing_information` entry with `field`,
`requirement_type`, `reason`, `blocks_content` (true only for a CORE_REQUIRED
gap, or a CONDITIONAL_REQUIRED gap a check confirms mandatory). Never invent a
value to close a gap.

## 4. Rule provenance — classification tags

Every recommendation this Skill emits carries **exactly one** tag:

| Tag | Meaning |
|---|---|
| **OFFICIAL** | Stated in Shopee official documentation (Open Platform docs, Centro de Educação do Vendedor, Central de Ajuda). Cite it. Until a source is read `LIVE`, an OFFICIAL rule still carries `⚠ verify`. |
| **DYNAMIC** | A value/rule that depends on the current category / shop / region and MUST be fetched at listing time (e.g. from `get_category`, `get_attributes`, `get_brand_list`, `get_item_limit`, `get_dts_limit`). **Never hardcoded.** On Shopee, `DYNAMIC` carries more weight than on Mercado Livre — almost every numeric limit is dynamic. |
| **INTERNAL** | Good practice created for our operation. Not a Shopee requirement. |
| **EXPERIMENTAL** | An unproven hypothesis (e.g. a ranking interpretation). Must be validated with data before it influences a hard decision. |
| **LEARNED** | Derived from our own historical performance data (future) — maps naturally to Shopee's penalty-point / shop-performance feedback. |
| **UNVERIFIED** | Asserted by non-primary evidence only, or an open question. Not a rule to rely on. Pair with a `dynamic_checks_required` entry. |

Never present INTERNAL / EXPERIMENTAL / LEARNED / UNVERIFIED as an official Shopee
rule. Official docs beat external sources; flag any conflict for human review.

**Verification quality is a separate axis from the tag.** Record one of
`LIVE` / `SEARCH_INDEXED` / `UNVERIFIED` on every rule. Source *authority* and
verification *quality* are independent — a rule can be `OFFICIAL` +
`SEARCH_INDEXED` (an official page reconstructed from search, not read live).
`references/official-sources.md` holds the registry.

## 5. Evidence model (product facts)

Classify every `ProductMaster`-derived fact:

| State | Meaning |
|---|---|
| **CONFIRMED** | Backed by a reliable provenance (spec sheet, manufacturer data, physical inspection). |
| **INFERRED** | Reasonably deduced; needs a sanity check before it drives a hard claim. |
| **MISSING** | No value. (A gap — different from a wrong value.) |
| **CONFLICTING** | Sources disagree → stop, flag for human review, never auto-resolve. |
| **UNSUPPORTED** | A value is present but has no backing source. The presence of a value is never itself evidence. |

Non-negotiable, and `SHARED_CORE_CANDIDATE`:

```
marketplace data      ≠ product truth
competitor listing    ≠ product truth
buyer review          ≠ product truth
a generated/edited image ≠ evidence for a newly introduced product fact
```

`claim strength ≤ evidence strength` — no invented certification, medical
efficacy, authenticity, licence, compatibility, safety or regulatory approval.
See `references/compliance.md`.

## 6. Shopee entity model (current best understanding)

**Provisional** — `SEARCH_INDEXED`; every API-contract detail `⚠ verify`. Fuller
treatment in `references/product-structure.md` and `references/variations.md`.

```
Shop  (shop_id, Shopee-assigned, region BR)          — the seller storefront
 └── Item  (item_id, Shopee-assigned)                — THE LISTING
      │     item_sku  (seller-controlled, mutable)   — seller-side identity
      ├── Tier Variation   (≤ 2 tiers, positional, no id; option list per tier;
      │                     tier-1 options may each carry an image)
      └── Model  (model_id, Shopee-assigned)         — THE SELLABLE VARIANT
                 model_sku (seller-controlled, mutable)
                 tier_index[]  — which option of each tier this model is
                 price, stock  — per model
```

| Entity | Meaning | Shopee id | Product Factory mapping | Verification |
|---|---|---|---|---|
| Shop | seller storefront on Shopee BR | `shop_id` | account/shop mapping | `SEARCH_INDEXED` |
| Item | the listing | `item_id` (called `product_id` in some v2 docs) | 1 ProductMaster → 1 Item per shop | `SEARCH_INDEXED`; API contract `⚠ verify` |
| Tier Variation | the item's variation axes | none (positional) | internal variation axes | `SEARCH_INDEXED`; caps `⚠ verify` |
| Model | one sellable combination of tier options | `model_id` + `tier_index[]` | internal `variant_id` / SKU ↔ `model_id` | `SEARCH_INDEXED`; API contract `⚠ verify` |
| Brand | brand attribute value | `brand_id` | — | `SEARCH_INDEXED` |

An item with **no** variation maps its single sellable unit to the `item_id`
(and `item_sku`).

### 6.1 Variation / model pattern (maps well to Product Factory)

Shopee appears to be a **combination / matrix** model:

```
Cor:     Preto, Branco
Tamanho: P, M, G
   → 6 sellable Models, each = one (Cor, Tamanho) combination
```

This maps cleanly to internal Product Factory variants at the **model** level.
But do **not** lock as stable constants: maximum tiers, maximum options per tier,
maximum model count, per-model SKU limits, images per tier. Those live in
`references/variations.md` as `DYNAMIC` / `UNVERIFIED` (resolve via
`get_item_limit`).

### 6.2 Operation-scoped execution (conceptual)

`EXECUTION_STATUS` (§7) is evaluated **for the target operation**. No formal
operation enum is fixed yet. Conceptual examples — exact API operation mapping is
`⚠ verify` until the Open Platform contract is confirmed:

`CREATE_ITEM`, `UPDATE_ITEM`, `INIT_VARIATION`, `UPDATE_PRICE`, `UPDATE_STOCK`,
`UPLOAD_MEDIA`, `UNLIST_ITEM`, `DELETE_ITEM`, `FETCH_ITEM`.

A create operation must **never** require an `item_id` / `model_id` that the
create itself returns.

### 6.3 Country scope

Primary marketplace: **Shopee Brasil (BR)**. Tag any rule whose origin is
region-specific:

| Scope tag | Meaning |
|---|---|
| `GLOBAL_API` | An Open Platform API shape believed common across regions. Still `⚠ verify` for BR. |
| `BRAZIL` | Confirmed (to the extent anything is) for the BR site. |
| `OTHER_MARKET_REFERENCE` | From Singapore / Malaysia / Thailand / Philippines / Vietnam / Taiwan. **Must not** silently become a Shopee Brasil rule. |

### 6.4 NOT imported from Mercado Livre

These are Mercado Livre concepts. Phase 01 found **no** Shopee equivalent. Do
**not** create Shopee analogues for symmetry; only reference them to state the
non-mapping:

`User Product`, `Family` / `family_name`, `PARENT_PK` / `CHILD_PK`,
`Multi Origem` (`STANDARD` / `MULTI_ORIGIN_SINGLE_WAREHOUSE` /
`MULTI_ORIGIN_MULTIWAREHOUSE`), `listing_allowed`, `EMPTY_GTIN_REASON`,
shared catalog product page / Buy Box, `POST /items/validate`, `/performance`,
`moderations/last_moderation`.

Whether Shopee BR has **any** catalogue / "produto" grouping concept is
**UNRESOLVED** (§13, gap G6). Safe default: every Shopee listing is standalone.

## 7. Readiness model — four independent dimensions

Marked `SHARED_CORE_CANDIDATE` — adopted here **provisionally**, not as Shopee
policy. Full detail and aggregation in `references/quality-audit.md`.

| Dimension | Question | States |
|---|---|---|
| **`CONTENT_STATUS`** | Can Product Factory truthfully represent the product? | `PASS` / `REVIEW` / `FAIL` |
| **`PUBLICATION_STATUS`** | Does the assembled listing satisfy currently **resolved** Shopee requirements? | `PASS` / `REVIEW` / `FAIL` |
| **`EXECUTION_STATUS`** | Can the target Shopee operation be performed now with this shop / API / context? Evaluated **per operation** (§6.2). | `PASS` / `REVIEW` / `FAIL` |
| **`QUALITY_STATUS`** | Is the listing complete and optimised beyond minimum validity? | `PASS` / `REVIEW` only |

- **Derived compatibility `status`** (not the source of truth — read the four
  dimensions): `FAIL` if any of CONTENT / PUBLICATION / EXECUTION is `FAIL`; else
  `REVIEW` if any dimension (including QUALITY) is `REVIEW`; else `PASS`.
- **Pending vs FAIL (safety semantics):**
  `not checked / unresolved → REVIEW`. `checked and confirms an incompatibility →
  FAIL`. **`unknown ≠ FAIL`.** A non-empty `dynamic_checks_required` is **never**
  an automatic `FAIL`.
- A `BLOCKER` finding forces its `affects[]` dimension(s) to `FAIL`; a `MAJOR`
  forces them to `REVIEW`. `QUALITY_STATUS` has no `FAIL` — a defect severe
  enough to make the listing misleading, unsafe or wrong routes to CONTENT /
  PUBLICATION; an operation-blocking marketplace state routes to EXECUTION.
- **A provisional rule being unresolved does not make every listing `REVIEW`.**
  Trigger a check only when it is **relevant** to the product at hand. Ordinary
  products are not held at `REVIEW` because a theoretical policy is unverified.

**Gates:** content generation → `CONTENT_STATUS = PASS`. Publication →
CONTENT + PUBLICATION + EXECUTION all `PASS`. `QUALITY_STATUS = REVIEW` may still
be acceptable. Any stricter bar is INTERNAL Product Factory policy.

## 8. Dynamic-check policy

Call the Shopee API / MCP (never guess) for every `DYNAMIC` value. If the
API/MCP is unavailable, each dependent decision becomes a `dynamic_checks_required`
entry — **pending**, holding the relevant dimension at `REVIEW`, never `FAIL`.

Every dynamic check is defined by: `check` · `why` · `source` (API/doc) ·
`pending effect` · `confirmed-incompatibility effect` · `verification`.

Conceptual example:

```
check:  resolve_shopee_category_requirements
state:  pending | executed
pending                          → PUBLICATION_STATUS = REVIEW
executed + compatible            → no blocker
executed + mandatory req. unmet  → PUBLICATION_STATUS = FAIL
```

The registry lives in `references/api-and-auth.md` (§"Dynamic check registry").
At minimum it covers: category requirements, attribute requirements, brand
requirement/value, identifier requirement, title limit, description limit, image
limits, variation limits, logistics requirements, shop/API capability, inventory
model, compliance applicability. **No endpoint is hardcoded** — each check names
a resource, and each resource carries its own `verification` state.

## 9. Listing creation workflow (high level)

Follow in order. Each step writes into the draft and may raise findings. Steps
that depend on an unverified Shopee rule produce a `dynamic_checks_required`
entry and a `REVIEW`, not a `FAIL`.

1. **Evidence + requirement validation** — classify every field by evidence
   (§5) and requirement layer (§3). `CONFLICTING` → stop, human review. Only a
   CORE_REQUIRED gap (or a confirmed-mandatory CONDITIONAL gap) blocks content.
2. **Product classification** — what it physically is; which traits matter.
3. **Category** — `discover → resolve → validate` (INTERNAL safe workflow,
   `references/categories.md`): predict candidates, confirm a **leaf**, check fit,
   then pull attributes / brand list / DTS limits for that leaf. Do not assume
   Shopee exposes a category predictor unless verified.
4. **Attributes** — resolve mandatory vs recommended for the category
   (`references/attributes.md`); map ProductMaster → attribute ids / values;
   never invent a value. Recommended-only attributes are `QUALITY`, not
   `PUBLICATION`.
5. **Brand & identifiers** — resolve brand (real + evidence-backed, or
   "Sem marca"); resolve identifier requiredness per category; never invent
   (`references/brand-and-identifiers.md`).
6. **Variation / model plan** — axes and each model's `tier_index`; keep the
   internal `variant_id` / SKU stable; write internal SKU into `model_sku`
   (`references/variations.md`). Flag anything beyond 2 tiers for review.
7. **Competitor / review intelligence** — optional; patterns and buyer
   objections only, never copied content, never treated as product fact or
   Shopee policy (`references/competitor-research.md`, `references/review-mining.md`).
8. **Title** — within the category's `DYNAMIC` length limit (the ≈255/256
   discovery value is **provisional, not enforced**); Title Case; no
   diversion / spam / promo terms; brand/model/spec only where they legitimately
   exist (`references/titles-and-seo.md`).
9. **Description** — no external links / contact info; structured and scannable;
   claims evidence-backed; the ≈5,000-char discovery value is **provisional**
   (`references/descriptions.md`).
10. **Images** — keep the layers of `references/images.md` separate: (A) Shopee
    official requirements, (B) Shopee official recommendations, (C) INTERNAL
    gallery strategy, (D) Product Identity Guard. Do not lock 1–9 / 1:1 / 3:4 /
    ≥60% / cover-text-ban as final constants — each carries its discovery
    evidence and verification state. Run the identity audit on every
    generated/edited asset.
11. **Video** — listing video only; Shopee Video and Shopee Live are separate
    surfaces with their own policies (`references/video.md`). Every media rule
    carries a `scope`.
12. **Pricing** — separate product fact vs commercial context vs publication
    requirement vs execution mechanism (`references/pricing.md`). No hardcoded
    min/max, gap or discount rules.
13. **Inventory** — model-level stock where models exist; treat the BR
    warehouse model as **unresolved** (`references/inventory.md`). Do not import
    Multi Origem.
14. **Logistics** — weight, dimensions, channels, handling time, pre-order,
    dangerous goods — placeholders to resolve (`references/logistics.md`).
15. **Return prevention** — run the checklist and the mandatory ambiguity
    question (`references/return-prevention.md`).
16. **Compliance** — resolve prohibited / restricted / regulated status as a
    *procedure*, not a frozen list (`references/compliance.md`); claims safe;
    no contact / external-diversion content. Findings feed CONTENT / PUBLICATION
    / EXECUTION.
17. **Moderation / enforcement context** — post-publication only; penalty-point
    and item-status state kept separate from ProductMaster truth
    (`references/moderation-and-enforcement.md`).
18. **Evaluate readiness** — the four dimensions (§7, `references/quality-audit.md`).
19. **READY FOR REVIEW** — emit draft + audit JSON. Stop.

## 10. Reference routing

| Question you are answering | Read |
|---|---|
| Which sources exist, how well verified, what API resources are believed to exist | `references/official-sources.md`, `references/api-and-auth.md` |
| Shop / Item / Tier Variation / Model; what field holds what; id mapping | `references/product-structure.md` |
| Category discover → resolve → validate; leaf requirement; open questions | `references/categories.md` |
| Mandatory vs recommended attributes; regulated attributes; value semantics | `references/attributes.md` |
| Brand requirement / "Sem marca" / registration; GTIN/EAN conditional | `references/brand-and-identifiers.md` |
| Tier/option/model caps; combination semantics; post-sale mutability | `references/variations.md` |
| Title length (DYNAMIC), Title Case, policy vs recommendation vs SEO hypothesis | `references/titles-and-seo.md` |
| Description limit (DYNAMIC), format, prohibited content | `references/descriptions.md` |
| Image specs / recommendations / gallery strategy / Product Identity Guard | `references/images.md` |
| Listing video vs Shopee Video vs Shopee Live — scope separation | `references/video.md` |
| Price fields, `price_limit`, misleading-price policy | `references/pricing.md` |
| Stock level, warehouse model (unresolved), update semantics | `references/inventory.md` |
| Weight / dimensions / channels / handling time / pre-order / dangerous goods | `references/logistics.md` |
| Prohibited / restricted / regulated resolution; claims safety; IP/brand; contact-diversion | `references/compliance.md` |
| `item_status` graph; penalty points; appeals; API-exposure gaps | `references/moderation-and-enforcement.md` |
| Readiness dimensions, aggregation, gates, output JSON, sub-scores | `references/quality-audit.md` |
| Reducing wrong buyer expectations before finishing | `references/return-prevention.md` |
| Analysing competitor listings without copying | `references/competitor-research.md` |
| Mining buyer reviews of similar products | `references/review-mining.md` |
| Auth model, endpoint registry, BR access risk, dynamic-check registry | `references/api-and-auth.md` |

## 11. Output / audit contract

Emit the listing draft **and** this JSON. Product Factory-internal semantics
(mirrors the Mercado Livre Skill's shape as a *second marketplace*, not a copy):

```json
{
  "marketplace": "shopee_br",
  "audit_mode": "PRE_PUBLICATION | POST_PUBLICATION",
  "content_status": "PASS | REVIEW | FAIL",
  "publication_status": "PASS | REVIEW | FAIL",
  "execution_status": "PASS | REVIEW | FAIL",
  "execution_operation": "the target operation EXECUTION_STATUS was evaluated for (free text, e.g. CREATE_ITEM, UPDATE_STOCK)",
  "quality_status": "PASS | REVIEW",
  "status": "PASS | REVIEW | FAIL",
  "status_note": "derived compatibility status — read the four dimensions instead",
  "scores": {
    "product_accuracy": 0, "category": 0, "attributes": 0, "brand_identifiers": 0,
    "title_seo": 0, "description": 0, "images": 0, "variations": 0,
    "pricing": 0, "inventory_logistics": 0, "consistency": 0,
    "return_prevention": 0, "compliance": 0
  },
  "blockers": [],
  "major": [],
  "warnings": [],
  "recommendations": [],
  "compliance_findings": [],
  "missing_information": [],
  "dynamic_checks_required": [],
  "sources_used": []
}
```

Each finding: `{ "code", "severity": "BLOCKER|MAJOR|WARNING|RECOMMENDATION",
"affects": ["CONTENT"|"PUBLICATION"|"EXECUTION"|"QUALITY"], "rule_tag":
"OFFICIAL|DYNAMIC|INTERNAL|EXPERIMENTAL|LEARNED|UNVERIFIED", "verification":
"LIVE|SEARCH_INDEXED|UNVERIFIED", "issue", "evidence", "fix", "source" }`.
`BLOCKER` → its `affects[]` dimension(s) `FAIL`; `MAJOR` → `REVIEW`. Every
`BLOCKER` states what failed, `affects`, evidence/source and a remedy; every
`REVIEW` states what resolves it.

Each `compliance_finding`: `{ "rule", "source", "classification":
"OFFICIAL|DYNAMIC|INTERNAL|UNVERIFIED", "scope", "evidence", "status":
"PASS|REVIEW|FAIL", "affects": ["CONTENT"|"PUBLICATION"|"EXECUTION"], "remedy" }`.

Each `missing_information`: `{ "field", "requirement_type":
"CORE_REQUIRED|CONDITIONAL_REQUIRED|PUBLICATION_REQUIRED|COMMERCIAL_OPTIONAL",
"reason", "blocks_content": true|false }`.

Each `dynamic_checks_required`: `{ "check", "why", "source", "pending_effect",
"confirmed_incompatibility_effect", "verification" }`.

Any change to the dimension list, status states, severity scale, aggregation rule
or finding shape must be mirrored across this file and
`references/quality-audit.md` (and, once stable, the repo `CLAUDE.md`).

## 12. Freshness policy

- Bump `last_reviewed` here and in each reference file whenever it is re-verified.
- Every Shopee-specific rule starts life `⚠ verify`. Remove the marker for a rule
  **only** after a maintainer reads the live primary source (or the MCP/API layer
  calls the endpoint and records the real response) and updates the row's
  `verification` to `LIVE`.
- Re-verify the volatile files at least quarterly: `product-structure`,
  `variations`, `categories`, `attributes`, `brand-and-identifiers`, `images`,
  `pricing`, `inventory`, `logistics`, `api-and-auth`.

## 13. Known discovery gaps

These block **locking** rules and building any execution client. They do **not**
block scaffolding or drafting. They should shrink as later phases resolve them.

| # | Gap | Consequence | Resolve in |
|---|---|---|---|
| **G1** | No readable primary Shopee Open Platform API contract (portal unreachable; not usefully indexed). | Every endpoint / field / payload / limit is `SEARCH_INDEXED` or `UNVERIFIED`. `PUBLICATION_STATUS` hard checks cannot be trusted yet. | Phase 02.2 |
| **G2** | Whether the Open Platform API is available to Brazil-domiciled partners / shops at all is **unresolved**. | The entire `EXECUTION` layer and any future publish pipeline depend on it. Output stays publish-agnostic. | Phase 02.3 |
| **G3** | Core numeric limits (title ≈255/256, description ≈5,000, images 1–9, image dims ≈350×350 / ≈1024, tier/option/model caps, price/stock bounds, days-to-ship) are only MEDIUM/LOW confidence. | None is locked. All are `DYNAMIC` via `get_item_limit` / `get_dts_limit` / category lookups; the "≈" numbers live only in prose as provisional. | Phase 02.4 |
| **G4** | BR stock / warehouse model unresolved — single seller pool vs multi-warehouse `location_id`; absolute vs incremental stock writes; concurrency behaviour. | `inventory.md` stays conservative; Multi Origem is **not** imported. | Phase 02.5 |
| **G5** | No dedicated pre-publication validator has been confirmed (no `POST /items/validate` analogue found — but the negative is not proven either). | `PUBLICATION_STATUS` must lean on up-front limit/attribute fetches + local payload checks; the `add_item` response is treated as the authoritative gate. | Phase 02.2 / 02.5 |
| **G6** | Catalogue / "produto" grouping concept unresolved — no evidence of a shared product page / Buy Box, but not confirmed absent. | Every listing treated as standalone; no catalogue-association logic built; no `catalog.md` created. | Phase 02.10 |
| **G7** | Penalty-point mechanics (weekly cadence, 60-day validity, threshold values) and any BR violations / account-health API are unverified. | Enforcement treated as Seller-Center-only and separate from ProductMaster truth. | Phase 02.8 |
| **G8** | Listing-video constraints (duration / size / aspect) vs Shopee Video / Shopee Live constraints not separated with confidence. | Only state Shopee Video / Live numbers with an explicit `scope`; listing-video specs marked unknown. | Phase 02.7 |

## Sources

- Phase 01 discovery report — `research/shopee-listing-skill/discovery-report.md`
  — internal — consulted 2026-08-28 — the research map this scaffold is built
  from; records the `SEARCH_INDEXED` / `UNVERIFIED` posture, the entity/API/
  dynamic-check tables, the Mercado Livre comparison and the six+ discovery gaps.
- Mercado Livre Skill — `.claude/skills/mercado-livre-listing-best-practices/` —
  internal — read directly — **architectural reference only**, not a Shopee rule
  source. Provenance tags, the four-dimension readiness model, evidence model,
  requirement layers, Product Identity Guard, claim-safety, compliance-as-
  procedure and the audit contract shape are adapted here as Shopee-worded rules.
- No Shopee source has been read `LIVE`. Per-topic source rows (all
  `SEARCH_INDEXED` or `UNVERIFIED`) are in each reference file's `## Sources`
  block and consolidated in `references/official-sources.md`.
