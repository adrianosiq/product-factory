# Attributes — product fact vs Shopee category requirement

last_reviewed: 2026-08-28
volatile: true
classification: OFFICIAL (model) — verification SEARCH_INDEXED; value/unit semantics `⚠ verify`

## 1. Keep two things separate

| Concept | Question | Source |
|---|---|---|
| **product fact** | Is this attribute value true for the real product, and how do we know? | `ProductMaster` + evidence model (SKILL.md §5) |
| **Shopee category requirement** | Does the chosen category require this attribute to publish? | `get_attributes` for the leaf (`DYNAMIC`) |

A value being *required by the category* never licenses inventing it. A value
being *true* never means the category will accept it in that form.

## 2. What discovery found (all `SEARCH_INDEXED`, `⚠ verify`)

| Aspect | Finding | Tag | Verification |
|---|---|---|---|
| Endpoint | `get_attributes`, per `category_id` | OFFICIAL | `SEARCH_INDEXED` |
| Fields | `attribute_id`, `original_attribute_name`, `is_mandatory` (bool), `input_type` (`DROP_DOWN` / `MULTIPLE_SELECT` / `TEXT_FILLING` / `COMBO_BOX` / …), `format_type` (`NORMAL` / `QUANTITATIVE`), `attribute_value_list` (`value_id`, `original_value_name`, `value_unit`), `date_format` | OFFICIAL | `SEARCH_INDEXED`; exact enum set `UNVERIFIED` |
| Requiredness | **static per category**, expressed by `is_mandatory`. No separate conditional-resolution endpoint found (no analogue of ML's `POST .../attributes/conditional`). | OFFICIAL | `SEARCH_INDEXED` |
| Free-text vs value-id | both, per `input_type` — `TEXT_FILLING` allows free text; dropdowns need a `value_id` | OFFICIAL | `SEARCH_INDEXED` |
| Units | `value_unit` on quantitative attributes | OFFICIAL | `SEARCH_INDEXED` |
| Regulatory fields | appear as **ordinary attributes** in regulated categories (INMETRO number, ANVISA registration, …) — not a separate subsystem | OFFICIAL | `SEARCH_INDEXED` |

## 3. Classification for the Skill

- **Attribute requiredness is `DYNAMIC`** — always fetch `get_attributes` for the
  resolved leaf; never hardcode which attributes a category needs.
- **Mandatory (`is_mandatory = true`) missing** → `PUBLICATION_STATUS = FAIL`
  after the category is resolved.
- **Recommended / "quality" attributes** → `QUALITY_STATUS`, never
  `PUBLICATION_STATUS` (same split the ML Skill makes for technical specs).
- **Check pending** (category or `get_attributes` not yet resolved) →
  `PUBLICATION_STATUS = REVIEW`, `dynamic_checks_required:
  resolve_category_attributes`.

## 4. Open questions

- Whether requiredness can ever depend on **other field values** or on the
  **shop** (conditional / shop-specific requiredness). No mechanism found —
  `UNVERIFIED`.
- Exact `input_type` / `format_type` enum sets — `UNVERIFIED`.
- `value_id` vs free-text rules per `input_type`; unit handling on quantitative
  attributes — `⚠ verify`.
- Whether regulated-category attributes (INMETRO / ANVISA numbers) are validated
  by Shopee or merely stored — `UNVERIFIED`. See `references/compliance.md`.
- Brand is handled separately — `references/brand-and-identifiers.md`.

## 5. Dynamic-check placeholder

```
check: resolve_category_attributes
why:   determine which attributes are mandatory (is_mandatory) vs recommended
source: get_attributes for the resolved leaf   ← endpoint NOT hardcoded; see api-and-auth.md §3
pending:                REVIEW (PUBLICATION)
executed + all mandatory filled: no blocker
executed + a mandatory attribute unmet: FAIL (PUBLICATION)
verification: SEARCH_INDEXED
```

## Sources

- `get_attributes` fields, `is_mandatory`, `input_type`, quantitative units —
  `github.com/wjp-letgo/shopeego`, `github.com/teacat/shopeego` — community SDKs
  — consulted 2026-08-28 — `SEARCH_INDEXED` (enum sets `UNVERIFIED`).
- Regulatory fields as ordinary attributes — `help.shopee.com.br` art. 76226 —
  Central de Ajuda — consulted 2026-08-28 — `SEARCH_INDEXED`.
- Required-vs-quality split — `.claude/skills/mercado-livre-listing-best-practices/references/quality-audit.md`
  — internal — architectural reference only.
- Full detail: `research/shopee-listing-skill/discovery-report.md` §5.
