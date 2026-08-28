# Quality audit

last_reviewed: 2026-08-27

Final gate. Produces the structured output in `SKILL.md` §9. No publishing.

## Severity scale

| Severity | Meaning | Effect on status |
|---|---|---|
| **BLOCKER** | Wrong/unsafe/dishonest, or a hard ML rule broken. | status = FAIL |
| **CRITICAL** | Serious quality/compliance gap; listing would underperform or risk review. | status = REVIEW (at best) |
| **WARNING** | Real issue, not disqualifying. | status may still be PASS |
| **RECOMMENDATION** | Improvement idea. | no status effect |

## Status rule

- `FAIL` — any BLOCKER, or unresolved `dynamic_checks_required`.
- `REVIEW` — no BLOCKER but ≥1 CRITICAL, or `missing_information` affecting accuracy.
- `PASS` — no BLOCKER, no CRITICAL, all DYNAMIC checks resolved, evidence clean.

## Dimensions (score 0–100 each)

| Dimension | Pass bar | Key checks (see linked file) |
|---|---|---|
| `PRODUCT_ACCURACY` | No invented facts; all values CONFIRMED/vetted-INFERRED | `attributes.md` §evidence |
| `CATEGORY` | Confirmed via predictor; domain correct | `categories.md` |
| `CATALOG` | Catalog checked; link is exact match or independent is justified | `catalog.md` |
| `FAMILY_NAME_TITLE` | Right flow; DYNAMIC length; brand/model consistent; no prohibited words | `titles-and-family-name.md` |
| `ATTRIBUTES` | required/new_required/conditional filled; `value_id`s; GTIN real/"N/A" | `attributes.md`, `categories.md` |
| `SEARCH_RELEVANCE` | Filterable attributes filled; terms early; no stuffing | `seo-and-discovery.md` |
| `DESCRIPTION` | `plain_text` valid; complements ficha; no banned content; claims backed | `descriptions.md` |
| `IMAGES` | OFFICIAL limits; DYNAMIC count; identity unaltered; variant-correct | `images.md` |
| `VARIANTS` | Correct model; PARENT_PK/CHILD_PK right; per-variant stock/img; cap respected | `variations-and-user-products.md` |
| `CONSISTENCY` | No contradictions across the chain below | this file |
| `RETURN_PREVENTION` | Checklist clean; mandatory question answered "no" | `return-prevention.md` |
| `COMPLIANCE` | Condition rules, prohibited claims, IP, regulated-category needs | `descriptions.md`, `pricing-and-commercial.md` |

## Mandatory cross-consistency check

Compare the same fact everywhere it appears:

```
ProductMaster ⇄ Category ⇄ Catalog ⇄ family_name/Title ⇄ Attributes ⇄ Description ⇄ Images ⇄ Variants
```

- Any mismatch of a material fact (material, model, quantity, color, inclusion) →
  **BLOCKER — PRODUCT DATA CONFLICT**.
  - Example: ProductMaster `Material = Acetato`, description says `TR90` →
    `BLOCKER — PRODUCT DATA CONFLICT (material)`.
- Cosmetic wording differences that don't change meaning → WARNING or none.

## Per-finding shape

```json
{
  "dimension": "IMAGES",
  "severity": "BLOCKER",
  "issue": "Variant 'Preto' hero image shows the blue unit",
  "evidence": "image img_003 vs attribute COLOR=Preto",
  "fix": "Replace img_003 with a photo of the black unit",
  "rule_tag": "INTERNAL",
  "source": "references/images.md"
}
```

## Output

Emit the `SKILL.md` §9 JSON plus the listing draft (model, `category_id`,
`family_name`/title, attributes map, description `plain_text`, variant table,
image plan with roles + variant mapping). Populate:

- `dynamic_checks_required` — every DYNAMIC value not confirmed via API this run.
- `missing_information` — every MISSING ProductMaster field that mattered.
- `sources_used` — the reference files and any live docs/API endpoints consulted.

Then stop at **READY FOR REVIEW**.

## Sources

- Validador de publicações (`POST /items/validate`) — https://developers.mercadolivre.com.br/pt_br/validador-de-publicacoes — Developers — verified 2026-08-27 (search-indexed copy; live page returns 403 to bots) — `POST /items/validate` with the listing payload; HTTP 204 when no problems are found, HTTP 400 with a `cause[]` array of errors/warnings otherwise. Optional, meant for testing; passing it is a technical pre-check only, not a guarantee that publication will succeed or that the content is correct.
- Validações — https://developers.mercadolivre.com.br/pt_br/validacoes — Developers — ⚠ verify — consulted 2026-08-27 — validation flow and seller correction requests.
- All other checks derive from the linked reference files.
