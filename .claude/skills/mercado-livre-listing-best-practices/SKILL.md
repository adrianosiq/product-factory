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

Minimum `ProductMaster` fields the agent needs before starting:

- Product type / what the product physically is
- Brand, model / reference, manufacturer part number (if any)
- GTIN / EAN / UPC (or an explicit "does not exist / does not apply")
- Key physical attributes: material, dimensions, weight, color(s), capacity, power, etc.
- Variation axes (color, size, voltage, …) and the SKU list with per-variant stock
- What is included in the box / what is NOT included
- Compatibility data (for auto parts, accessories, electronics)
- Condition (new / used / refurbished)
- Source images (real photos of the real product / variant)
- Commercial inputs: cost, target price, listing type intent, shipping mode, availability lead time
- Any certifications, warranty terms, legal/regulatory notes

If a required field is missing, record it in `missing_information` and mark the
affected audit dimension — do not invent it.

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
| Title rules; `family_name` when title is generated | `references/titles-and-family-name.md` |
| How products get found; what is OFFICIAL vs strategy | `references/seo-and-discovery.md` |
| Ficha técnica, GTIN, "não se aplica", structured-first | `references/attributes.md` |
| Image formats, resolution, per-category limits, per-variant images | `references/images.md` |
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

- `category_id` via the category predictor
- `GET /categories/$CATEGORY_ID/attributes` — attribute ids, value lists, tags
  (`required`, `new_required`, `conditional_required`), `PARENT_PK`/`CHILD_PK`
- `GET /categories/$CATEGORY_ID/attributes/conditional` — conditional requirements
- category settings: title length limit, `max_pictures_per_item`,
  `max_pictures_per_item_var`, variation rules, `buying_mode`, listing types
- catalog match: does a `catalog_product_id` already exist for this product
- pre-publish validator endpoint for the assembled payload

If the API/MCP is unavailable, every dependent decision goes into
`dynamic_checks_required` and the audit cannot return `PASS`.

## 6. Listing creation workflow

Follow in order. Each step writes into the draft and can raise audit findings.

1. **Evidence validation** — classify every ProductMaster field as CONFIRMED /
   INFERRED / MISSING / CONFLICTING / UNSUPPORTED (`references/attributes.md` §evidence).
   CONFLICTING between sources → stop and flag for human review.
2. **Product classification** — what is it, physically; which domain-relevant traits matter.
3. **Category / domain** — run the predictor; confirm the domain; pull the attribute set (DYNAMIC).
4. **Catalog check** — is there a `catalog_product_id` match? If yes, decide
   associate-to-catalog vs independent listing before writing any content (`references/catalog.md`).
5. **Dynamic attributes** — resolve required / new_required / conditional_required;
   map ProductMaster → attribute ids and `value_id`s; identify variation axes
   (`PARENT_PK` / `CHILD_PK`).
6. **Competitor / search intelligence** — optional, if tools available
   (`references/competitor-research.md`, `references/review-mining.md`). Extract
   patterns and buyer objections; never copy content.
7. **Listing strategy** — Item vs User Product model; how many variants; listing type intent.
8. **`family_name` / title strategy** —
   - Title-is-provided flow: build the title per `references/titles-and-family-name.md` (OFFICIAL rules), respecting the category's DYNAMIC length limit.
   - Generated-title flow (User Products / UPtin): do NOT craft a title; optimize
     `family_name`, attributes and domain instead.
9. **Attributes** — fill structured fields first; never invent GTIN/brand/model/spec;
   use "não se aplica" only where legitimately applicable.
10. **Description** — `plain_text`; complements the ficha técnica; structured
    sections; no keyword stuffing, no unproven claims, no invented features.
11. **Variant strategy** — model the variants per `references/variations-and-user-products.md`;
    per-variant stock, images and (new model) sale conditions/price.
12. **Image strategy** — OFFICIAL constraints from `references/images.md` (DYNAMIC
    quantity), then the INTERNAL gallery plan; each image has a commercial job;
    the correct variant image maps to the correct variant; AI edits must not alter
    real product identity.
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

## 8. Blocking criteria (status = FAIL)

- Any `BLOCKER` finding (e.g. product-data conflict between fields, wrong-variant image, invented GTIN/spec).
- Category not confirmed via predictor, or required/new_required attributes unresolved.
- Title crafted for a flow where the title is ML-generated, or a hardcoded limit used where a DYNAMIC one exists.
- Images violating an OFFICIAL constraint, or exceeding the category's DYNAMIC max.
- Unanswered "reasonable misinterpretation" question from return prevention.
- `dynamic_checks_required` is non-empty and unresolved.

`REVIEW` = no blockers but ≥1 CRITICAL or missing information that affects accuracy.
`PASS` = no blockers, no CRITICAL, all DYNAMIC checks resolved.

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
