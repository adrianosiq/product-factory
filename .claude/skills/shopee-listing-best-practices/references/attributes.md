# Attributes — product fact vs Shopee category requirement

last_reviewed: 2026-08-28
phase_02_3_reviewed: 2026-09-03
phase_02_2_reviewed: 2026-08-30
volatile: true
classification: OFFICIAL (model) — verification SEARCH_INDEXED; value/unit semantics `⚠ verify`
phase_02_3_note: >-
  PRIMARY (SPD-023 get_attribute_tree). `GET /api/v2/product/get_attribute_tree`,
  request `category_id_list` (**max 20**) + `language` (BR: `pt-BR` / `en`).
  **The requiredness field is `mandatory` (bool)** — CORRECTS the `is_mandatory`
  guess (that name is `add_item` prose / `get_brand_list`). It is a **static
  per-category boolean** — no conditional-requiredness mechanism. `input_type`:
  SINGLE_DROP_DOWN=1, SINGLE_COMBO_BOX=2, FREE_TEXT_FILED=3, MULTI_DROP_DOWN=4,
  MULTI_COMBO_BOX=5. `format_type`: NORMAL=1, QUANTITATIVE_WITH_UNIT=2. Also
  `input_validation_type` (0-4), `date_format_type` (0-1), `attribute_unit_list`,
  `max_value_count`, `is_oem`, `support_search_value` (→ `search_attribute_value_list`,
  page not in corpus), `child_attribute_list` (recursive). `add_item`: for
  custom values set `value_id = 0` + `original_value_name`. Recommended (non-
  mandatory) attributes: `get_recommend_attribute` → QUALITY only.
phase_02_2_note: >-
  Resource name corrected `get_attributes` → `get_attribute_tree`; recommended
  set has its own resource `get_recommend_attribute`. Field names remain
  STILL_UNVERIFIED. See `research/shopee-api-contract/phase-02.2-report.md` §12.

## 1. Keep two things separate

| Concept | Question | Source |
|---|---|---|
| **product fact** | Is this attribute value true for the real product, and how do we know? | `ProductMaster` + evidence model (SKILL.md §5) |
| **Shopee category requirement** | Does the chosen category require this attribute to publish? | `get_attribute_tree` for the leaf (`DYNAMIC`) |

A value being *required by the category* never licenses inventing it. A value
being *true* never means the category will accept it in that form.

## 2. What discovery found (all `SEARCH_INDEXED`, `⚠ verify`)

| Aspect | Finding | Tag | Verification |
|---|---|---|---|
| Endpoint | **`get_attribute_tree`**, per `category_id` + `language` (Phase 02.2 **correction** — Phase 01/02.1 said `get_attributes`; the current method name in two independent SDKs is `get_attribute_tree`). A separate **`get_recommend_attribute`** (`category_id`, `item_name`) returns the recommended / quality set. | OFFICIAL | `SEARCH_INDEXED` (name MEDIUM; `get_attributes` may be an older alias) |
| Fields | believed: `attribute_id`, `original_attribute_name`, `is_mandatory` (bool), `input_type` (`DROP_DOWN` / `MULTIPLE_SELECT` / `TEXT_FILLING` / `COMBO_BOX` / …), `format_type` (`NORMAL` / `QUANTITATIVE`), `attribute_value_list` (`value_id`, `original_value_name`, `value_unit`), `date_format` | OFFICIAL | **`STILL_UNVERIFIED`** — the `get_attribute_tree` schema has not been read; field names are Phase 01 SDK reconstructions |
| Requiredness | **static per category**, believed expressed by `is_mandatory`. No separate conditional-resolution endpoint found (no analogue of ML's `POST .../attributes/conditional`). | OFFICIAL | `SEARCH_INDEXED` |
| Free-text vs value-id | both, per `input_type` — `TEXT_FILLING` allows free text; dropdowns need a `value_id` | OFFICIAL | `SEARCH_INDEXED` |
| Units | `value_unit` on quantitative attributes | OFFICIAL | `SEARCH_INDEXED` |
| Regulatory fields | appear as **ordinary attributes** in regulated categories (INMETRO number, ANVISA registration, …) — not a separate subsystem | OFFICIAL | `SEARCH_INDEXED` |

## 3. Classification for the Skill

- **Attribute requiredness is `DYNAMIC`** — always fetch `get_attribute_tree` for
  the resolved leaf; never hardcode which attributes a category needs.
- **Mandatory (`is_mandatory = true`) missing** → `PUBLICATION_STATUS = FAIL`
  after the category is resolved.
- **Recommended / "quality" attributes** → `QUALITY_STATUS`, never
  `PUBLICATION_STATUS` (same split the ML Skill makes for technical specs).
- **Check pending** (category or `get_attribute_tree` not yet resolved) →
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
source: get_attribute_tree for the resolved leaf  (recommended set: get_recommend_attribute)
        ← endpoint NOT hardcoded; see api-and-auth.md §3
pending:                REVIEW (PUBLICATION)
executed + all mandatory filled: no blocker
executed + a mandatory attribute unmet: FAIL (PUBLICATION)
verification: SEARCH_INDEXED (method name); field names STILL_UNVERIFIED
```

## Sources

- Resource name `get_attribute_tree` (+ `get_recommend_attribute`,
  `search_attr_value`) — `github.com/QuoVadis86/shopee-sdk`,
  `github.com/congminh1254/shopee-sdk` (`docs/managers/product.md`) — community
  SDKs — consulted 2026-08-28 — `SEARCH_INDEXED`, MEDIUM (method name);
  `phase-02.2-report.md` §12, §29 (C1).
- Field names (`is_mandatory`, `input_type`, quantitative units) —
  `github.com/wjp-letgo/shopeego`, `github.com/teacat/shopeego` — community SDKs
  — consulted 2026-08-28 — **`STILL_UNVERIFIED`** (schema not read).
- Regulatory fields as ordinary attributes — `help.shopee.com.br` art. 76226 —
  Central de Ajuda — consulted 2026-08-28 — `SEARCH_INDEXED`.
- Required-vs-quality split — `.claude/skills/mercado-livre-listing-best-practices/references/quality-audit.md`
  — internal — architectural reference only.
- Full detail: `research/shopee-listing-skill/discovery-report.md` §5.
