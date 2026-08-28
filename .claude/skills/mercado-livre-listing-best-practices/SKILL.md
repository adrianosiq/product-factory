---
name: mercado-livre-listing-best-practices
description: >-
  Create, review and optimize Mercado Livre Brasil listings from a structured
  ProductMaster. Turns product data into technically correct, complete,
  discoverable, honest and commercially competitive listings that respect the
  current Mercado Livre model (Items, User Products, Families, Catalog) and
  minimize the gap between listing expectation and delivered product. Produces a
  listing draft plus a structured quality audit. Never publishes.
last_reviewed: 2026-08-27
review_owner: product-create team
---

# Mercado Livre — Listing Best Practices

## 1. What this Skill is for

Use this Skill when an agent must transform a structured `ProductMaster` into a
**high-quality Mercado Livre Brasil listing draft**, or audit an existing draft.

The goal is NOT "the prettiest listing". The goal is:

> The clearest, most complete, most competitive and most faithful listing
> possible for the real product, respecting Mercado Livre's current structure and
> reducing the distance between the expectation the listing creates and the
> product the buyer receives.

When conversion and product accuracy conflict, **accuracy always wins.**

This Skill **never publishes**. It stops at `READY FOR REVIEW` and hands a draft +
audit to a human or to a separate publishing pipeline.

## 2. Required inputs

Not every `ProductMaster` field is needed to draft listing **content**. Sort each
field into one of four layers; the layer decides whether a gap blocks, needs
review, or is only a warning. Do not treat conditional or commercial fields as
universally mandatory — that creates false blockers.

### A. CORE_REQUIRED — identify and truthfully represent the product

A gap here is a **BLOCKER** for content creation.

- What the product physically is (type / function).
- The factual characteristics that distinguish *this* item from similar ones.
- Condition (new / used / refurbished) when it is not already unambiguous.
- When variants exist: the variation axes **and each variant's value on those
  axes**, plus a stable per-variant identifier (SKU or internal id) so variants
  are never mixed (`references/variations-and-user-products.md`). Per-variant
  media and stock belong to layers B/C, but the axis values that tell the
  variants apart are core.
- Evidence/source for every factual claim (Evidence rule below).

`brand` and `model` are **not** here: a genuine generic/unbranded product has no
commercial brand. They sit in layer B.

### B. CONDITIONAL_REQUIRED — required only when the context calls for it

Required *if and when applicable*. Resolve the real requirement dynamically
(category attributes, `POST /categories/$CATEGORY_ID/attributes/conditional`,
catalog rules, compatibility rules, logistics) — never as a global rule.

- Brand, model / reference, MPN — when the product is branded and/or the category
  requires them.
- Product identifier (GTIN / EAN / UPC / JAN / ISBN / ITF-14) — required, exempt
  or unsupported depending on category/product state; never a blanket global
  requirement. Identifier states, the `EMPTY_GTIN_REASON` absence mechanism and
  the evidence rules are in `references/attributes.md` §"Product identifiers".
- Dimensions, weight — when category attributes or the shipping mode require them;
  not "every ProductMaster must include dimensions".
- Compatibility data — for categories where compatibility is required/mandatory.
- Certifications, warranty terms, regulatory data — only when the category is
  regulated or the claim is being made. Never invented to fill the field.
- Non-axis variant attributes and per-variant media/stock — when variants exist
  (the distinguishing axis values themselves are CORE, layer A).
- "What is / isn't in the box" — when box contents are not obvious and a wrong
  assumption would mislead the buyer.
- Source images — required to **generate or edit product imagery** and to
  validate Product Identity; **not** required to draft a text-only listing from
  verified technical data.

### C. PUBLICATION_REQUIRED — not needed to draft, needed before publishing

A gap here does not block the draft; it is recorded and surfaced as a
publication-readiness gap (`REVIEW` at most), gated hard only at the separate
publish step.

- Price, currency, listing type.
- Stock / availability, handling / lead time.
- Shipping / logistics settings, seller-account context.
- Any marketplace field `POST /items/validate` requires that is not covered above.

### D. COMMERCIAL_OPTIONAL — useful for pricing/strategy, never required for content

Missing data here **never** blocks content creation. It produces a **WARNING**
and marks the matching analysis unavailable.

- Acquisition cost, landed cost.
- Target margin / contribution margin, internal profitability thresholds.
- Target price, competitor price targets, promotional strategy.

Missing acquisition cost → `WARNING — pricing/profitability analysis unavailable`,
never `BLOCKER — cannot create listing`.

### Recording gaps

For every gap add a `missing_information` entry carrying `field`,
`requirement_type` (`CORE_REQUIRED` | `CONDITIONAL_REQUIRED` |
`PUBLICATION_REQUIRED` | `COMMERCIAL_OPTIONAL`), `reason`, and `blocks_content`
(true only for a CORE_REQUIRED gap, or a CONDITIONAL_REQUIRED gap the
category/API confirms mandatory). Mark the affected audit dimension. Never invent
a value to close a gap.

**Evidence rule (unchanged):** a field being *absent* (`MISSING`) is different
from a field holding an *unsupported* claim (`UNSUPPORTED`) — no material value =
MISSING; `Material = TR90` with no source = UNSUPPORTED. Both are handled,
differently, per `references/attributes.md` §evidence. The presence of a value is
never by itself evidence.

## 3. Rule classification (use these tags everywhere)

Every recommendation this Skill emits MUST be attributable to one tag:

| Tag | Meaning |
|---|---|
| **OFFICIAL** | Stated in Mercado Livre official documentation (Developers, Central de Vendedores, Central de Ajuda). Cite the source. |
| **DYNAMIC** | Value/rule that depends on the current category/domain/site and MUST be fetched from the ML API at listing time. Never hardcode. |
| **INTERNAL** | Good practice created for our operation. Not an ML requirement. |
| **EXPERIMENTAL** | Commercial hypothesis, not yet proven. Must be validated with performance data. |
| **LEARNED** | Rule derived from our own historical performance data (future). |

Never present INTERNAL, EXPERIMENTAL or LEARNED as an official Mercado Livre rule.
If official docs and an external source disagree, the official doc wins and the
conflict is flagged for human review.

## 4. Reference map — read on demand, not all at once

| Question you are answering | Read |
|---|---|
| Which docs are authoritative, when were they last updated | `references/official-sources.md` |
| Item vs User Product vs Family vs Catalog; what field holds what | `references/product-structure.md` |
| Category prediction, domains, attribute tags, PARENT_PK/CHILD_PK | `references/categories.md` |
| Title mode (manual / generated / unresolved), `family_name`, brand/model in titles, OFFICIAL vs strategy | `references/titles-and-family-name.md` |
| How products get found; what is OFFICIAL vs strategy | `references/seo-and-discovery.md` |
| Ficha técnica, product identifiers (GTIN / `EMPTY_GTIN_REASON`), structured-first | `references/attributes.md` |
| OFFICIAL image specs vs INTERNAL gallery strategy vs Product Identity Guard; AI-edit safeguards | `references/images.md` |
| Description structure, plain_text, what to avoid | `references/descriptions.md` |
| Variations in the new model; migration from `variations[]` | `references/variations-and-user-products.md` |
| Catalog association, `catalog_product_id`, Buy Box | `references/catalog.md` |
| Price, listing types, shipping, requirement vs recommendation vs strategy | `references/pricing-and-commercial.md` |
| Analyzing competitor listings without copying | `references/competitor-research.md` |
| Mining reviews/questions of similar products | `references/review-mining.md` |
| Reducing wrong buyer expectations before finishing | `references/return-prevention.md` |
| Final audit dimensions, scoring, output JSON | `references/quality-audit.md` |

## 5. When to research live docs / call the API/MCP

**Research live official docs** (re-fetch the pages in `official-sources.md`) when:

- Any `source_last_updated` in a reference file is older than 90 days, OR
- The task touches User Products, families, variations, attributes, images,
  catalog, API limits or policies (the volatile areas), OR
- The reference file's rule is tagged `⚠ verify` (direct fetch was blocked when written).

**Call the ML API / MCP** (never guess) for every `DYNAMIC` value:

- `category_id` — **discover** a candidate (predictor
  `GET /sites/MLB/domain_discovery/search`, catalog, or a revalidated internal
  mapping) then **validate** it (`GET /categories/$CATEGORY_ID`: leaf +
  `settings.listing_allowed`; domain + attribute set fit the real product). The
  predictor is a discovery tool, not the sole authority
  (`references/categories.md` §1).
- `GET /categories/$CATEGORY_ID/attributes` — the **static** attribute model:
  attribute ids, value lists, tags (`required`, `new_required`,
  `conditional_required`, `catalog_required`), `PARENT_PK`/`CHILD_PK`; includes
  the product-identifier attribute (`GTIN`, …) and `EMPTY_GTIN_REASON` with its
  allowed `values[]`
- `POST /categories/$CATEGORY_ID/attributes/conditional` — resolves which
  `conditional_required` attributes are actually required **for this item**. The
  request body is the assembled item payload (category, price, condition,
  attributes, …); the evaluation depends on that data, so it is not a static
  category lookup. (OFFICIAL — verified 2026-08-27)
- `GET /categories/$CATEGORY_ID` — category `settings`: `max_title_length`,
  `max_pictures_per_item`, `max_pictures_per_item_var`, variation rules,
  `buying_mode`, listing types
- catalog match: does a `catalog_product_id` already exist for this product
- `POST /items/validate` — pre-publish technical validation of the assembled
  payload (HTTP 204 = no problems found; HTTP 400 = a `cause[]` list of
  errors/warnings). Optional; passing it is not a guarantee that publication
  will succeed or that the content is correct. (OFFICIAL — verified 2026-08-27)

If the API/MCP is unavailable, every dependent decision goes into
`dynamic_checks_required`. Those checks are **pending**, not failed: the audit is
`REVIEW` (not `FAIL`) and cannot reach `PASS` until they are executed (§8).

## 6. Listing creation workflow

Follow in order. Each step writes into the draft and can raise audit findings.

1. **Evidence + requirement validation** — classify every ProductMaster field by
   evidence (CONFIRMED / INFERRED / MISSING / CONFLICTING / UNSUPPORTED,
   `references/attributes.md` §evidence) and by requirement layer (§2 A–D).
   CONFLICTING between sources → stop and flag for human review. Only a
   CORE_REQUIRED gap — or a CONDITIONAL_REQUIRED gap confirmed mandatory by the
   category/API — blocks content creation; PUBLICATION_REQUIRED gaps are
   review-only and COMMERCIAL_OPTIONAL gaps are warnings.
2. **Product classification** — what is it, physically; which domain-relevant traits matter.
3. **Category / domain** — discover candidates (predictor / catalog / revalidated
   mapping), then **validate** the chosen one (leaf, `listing_allowed`, domain +
   attributes fit the real product); pull the attribute set (DYNAMIC). See
   `references/categories.md` §1.
4. **Catalog check** — is there a `catalog_product_id` match? If yes, decide
   associate-to-catalog vs independent listing before writing any content (`references/catalog.md`).
5. **Dynamic attributes** — resolve required / new_required / conditional_required;
   map ProductMaster → attribute ids and `value_id`s; identify variation axes
   (`PARENT_PK` / `CHILD_PK`).
6. **Competitor / search intelligence** — optional, if tools available
   (`references/competitor-research.md`, `references/review-mining.md`). Extract
   patterns and buyer objections; never copy content.
7. **Listing strategy** — Item vs User Product model; how many variants; listing type intent.
8. **`family_name` / title strategy** — first **detect the title mode**
   (`references/titles-and-family-name.md` §1); don't craft a `title` before it is
   known.
   - Manual / traditional flow: build the title per the OFFICIAL guidance, within
     the category's DYNAMIC `max_title_length`. No fixed 60; no "first 40
     characters" rule.
   - Generated / User Products flow (incl. UPtin): do NOT craft a title; optimize
     `family_name` (≤ the domain's `max_title_length`), attributes and domain.
   - Mode unresolved (account/marketplace context pending): record a dynamic
     check, don't assume a flow → `REVIEW`, not `FAIL`.
   - Brand/model/MPN go in the title/`family_name` only where they legitimately
     exist and are evidence-backed — never invented to fill a template.
9. **Attributes** — fill structured fields first; never invent a product
   identifier (GTIN/EAN/UPC/…), brand, model or spec. For a legitimate no-GTIN
   product use ML's `EMPTY_GTIN_REASON` mechanism (`references/attributes.md`),
   not a literal "N/A"; use "não se aplica" only for non-identifier attributes
   where it genuinely applies.
10. **Description** — `plain_text`; complements the ficha técnica; structured
    sections; no keyword stuffing, no unproven claims, no invented features.
11. **Variant strategy** — model the variants per `references/variations-and-user-products.md`;
    per-variant stock, images and (new model) sale conditions/price.
12. **Image strategy** — keep the three layers of `references/images.md` separate:
    (A) OFFICIAL ML rules — recommended specs vs hard limits, DYNAMIC per-category
    count, category-dependent cover-photo rules, moderation stays authoritative;
    (B) INTERNAL gallery plan — ~8–10 *useful* images is a guideline, not a
    quota; no filler; (C) **Product Identity Guard** — presentation may change,
    product identity may not. Run the post-generation identity audit on every
    generated/edited asset (`IDENTITY_PASS` / `IDENTITY_REVIEW` / `IDENTITY_FAIL`);
    a generated image is an output, never evidence.
13. **Return prevention** — run the checklist in `references/return-prevention.md`,
    including the mandatory ambiguity question.
14. **Compliance** — condition rules, prohibited claims, regulated-category needs,
    intellectual property on text and images.
15. **Quality audit** — run `references/quality-audit.md`.
16. **READY FOR REVIEW** — emit draft + audit JSON. Stop.

## 7. Audit workflow

Score each dimension 0–100 with issues, severity and recommendations:

`PRODUCT_ACCURACY`, `CATEGORY`, `CATALOG`, `FAMILY_NAME_TITLE`, `ATTRIBUTES`,
`SEARCH_RELEVANCE`, `DESCRIPTION`, `IMAGES`, `VARIANTS`, `CONSISTENCY`,
`RETURN_PREVENTION`, `COMPLIANCE`.

Cross-consistency check is mandatory (any mismatch is at least CRITICAL, a data
conflict is a BLOCKER):

```
ProductMaster ⇄ Category ⇄ Catalog ⇄ family_name/Title ⇄ Attributes ⇄ Description ⇄ Images ⇄ Variants
```

## 8. Status rules

An unresolved dynamic check has two distinct states; they resolve to different
statuses:

- **Pending** — the check has not been executed/resolved yet because marketplace
  context or API data is still missing. This is `REVIEW`, never `FAIL`.
- **Executed and failed** — the check ran and confirms a mandatory
  category/API/publication requirement the listing does not satisfy. This is a
  `BLOCKER` → `FAIL`.

### `FAIL` — any of:

- Any `BLOCKER` finding (e.g. product-data conflict between fields, wrong-variant image, an invented product identifier / spec).
- A **CORE_REQUIRED** ProductMaster gap (§2 A).
- A dynamic check that has been **executed** and confirms a mandatory requirement
  the listing does not satisfy — including a **CONDITIONAL_REQUIRED** field (§2 B)
  the category/API confirms mandatory and that is unmet.
- No **resolved** category, or a category whose validation was **executed** and
  shows it unusable/wrong (not a leaf, `listing_allowed = false`, or an attribute
  set that does not fit the product); validation ≠ "returned by the predictor"
  (`references/categories.md` §1). A category merely *pending* validation because
  ML data is still unavailable is REVIEW, not FAIL. Also: required/new_required
  attributes unresolved.
- A product identifier that is `REQUIRED_MISSING` — the category/API has
  **confirmed** it mandatory and there is neither a provenance-backed value nor
  an accepted `EMPTY_GTIN_REASON` (`references/attributes.md`). Requirement still
  `CONDITIONAL_PENDING` → REVIEW, not FAIL.
- A manual `title` crafted/sent in a flow where the current API **generates** it
  (title mode resolved and incompatible), a required generated-title /
  `family_name` mechanism violated, or an invented brand/model in the
  title/`family_name`. Title mode merely *unresolved* → REVIEW, not FAIL
  (`references/titles-and-family-name.md` §1).
- A hardcoded title/`family_name` length used where a DYNAMIC `max_title_length` exists.
- An image breaking a **confirmed hard** ML rule (unsupported format, below the
  accepted minimum, over the size cap, over the resolved category max count, or a
  cover-photo moderation rule for the category), or a **`IDENTITY_FAIL`** —
  geometry / colour / material / component / variant misrepresented, an invented
  feature, or an image contradicting ProductMaster (`references/images.md`).
  Missing an *only-recommended* spec (1200×1200, ~95%, RGB) is a WARNING;
  `IDENTITY_REVIEW` / commercial-gallery gaps are REVIEW/WARNING, not FAIL.
- Unanswered "reasonable misinterpretation" question from return prevention.

### `REVIEW` — no `FAIL` condition, but any of:

- ≥1 CRITICAL finding.
- `dynamic_checks_required` is non-empty because a check is still **pending**
  category/API context (not yet executed) — a CONDITIONAL_REQUIRED gap awaiting
  that context is here, not in `FAIL`.
- A **PUBLICATION_REQUIRED** gap (§2 C — a publication-readiness gap).

### `PASS` — all of:

- No `FAIL` condition and no CRITICAL finding.
- `dynamic_checks_required` is empty: every DYNAMIC check needed for publication
  has been executed and satisfied.
- Evidence clean.

Unresolved **COMMERCIAL_OPTIONAL** gaps (§2 D) stay as WARNINGs and never block
`PASS`. Missing ProductMaster data does not by itself cause `FAIL` when it is
PUBLICATION_REQUIRED or COMMERCIAL_OPTIONAL.

## 9. Output format

```json
{
  "marketplace": "mercado_livre",
  "status": "PASS | REVIEW | FAIL",
  "scores": {
    "product_accuracy": 0, "category": 0, "catalog": 0, "family_name_title": 0,
    "attributes": 0, "search_relevance": 0, "description": 0, "images": 0,
    "variants": 0, "consistency": 0, "return_prevention": 0, "compliance": 0
  },
  "blockers": [],
  "critical": [],
  "warnings": [],
  "recommendations": [],
  "missing_information": [],
  "dynamic_checks_required": [],
  "sources_used": []
}
```

Each finding: `{ "dimension", "severity": "BLOCKER|CRITICAL|WARNING|RECOMMENDATION", "issue", "evidence", "fix", "rule_tag": "OFFICIAL|DYNAMIC|INTERNAL|EXPERIMENTAL|LEARNED", "source" }`.

Each `missing_information` entry: `{ "field", "requirement_type": "CORE_REQUIRED|CONDITIONAL_REQUIRED|PUBLICATION_REQUIRED|COMMERCIAL_OPTIONAL", "reason", "blocks_content": true|false }` (§2). COMMERCIAL_OPTIONAL entries always have `blocks_content: false`.

Alongside the JSON, emit the listing draft: model (Item / User Product),
`category_id`, `family_name` and/or title, attributes (id → value), description
`plain_text`, variant table, image plan with per-image role and variant mapping.

## 10. Freshness policy

- Bump `last_reviewed` in each file whenever you re-verify it.
- Re-verify volatile files (`product-structure`, `variations-and-user-products`,
  `categories`, `attributes`, `images`, `catalog`, `pricing-and-commercial`) at
  least quarterly or before a large batch of listings.
- Any rule tagged `⚠ verify` was written without a successful direct fetch of the
  official page — confirm it against the live doc before relying on it.
