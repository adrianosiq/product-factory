# Attributes — product fact vs Shopee category requirement

last_reviewed: 2026-08-28
phase_02_3_reviewed: 2026-09-03
phase_02_2_reviewed: 2026-08-30
volatile: true
classification: OFFICIAL (model) — verification PRIMARY_VERIFIED (SPD-023 schema); per-category values are DYNAMIC
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
  HISTORICAL (superseded by phase_02_3_note above). Resource name corrected
  `get_attributes` → `get_attribute_tree`; recommended set has its own resource
  `get_recommend_attribute`. Field names were then STILL_UNVERIFIED — Phase 02.3
  read the schema (SPD-023): the requiredness field is `mandatory` (not
  `is_mandatory`). See `research/shopee-api-contract/phase-02.2-report.md` §12.

## 1. Keep two things separate

| Concept | Question | Source |
|---|---|---|
| **product fact** | Is this attribute value true for the real product, and how do we know? | `ProductMaster` + evidence model (SKILL.md §5) |
| **Shopee category requirement** | Does the chosen category require this attribute to publish? | `get_attribute_tree` for the leaf (`DYNAMIC`) |

A value being *required by the category* never licenses inventing it. A value
being *true* never means the category will accept it in that form.

## 2. What is known (Phase 02.3 primary — SPD-023)

| Aspect | Finding | Tag | Verification |
|---|---|---|---|
| Endpoint | **`GET /api/v2/product/get_attribute_tree`**, request `category_id_list` (**max 20**) + `language` (BR: `pt-BR` / `en`) → per-category `attribute_tree[]` (corrects the Phase 01/02.1 `get_attributes` guess). A separate **`get_recommend_attribute`** (`category_id`, `item_name`) returns the recommended / quality set. | OFFICIAL | `PRIMARY_VERIFIED` (SPD-023 / SPD-024) |
| Fields | `attribute_id`, `mandatory` (bool), `name`, `attribute_value_list[{value_id, name, value_unit, child_attribute_list}]`, `attribute_info{input_type (1–5), input_validation_type (0–4), format_type (NORMAL=1 / QUANTITATIVE_WITH_UNIT=2), date_format_type (0–1), attribute_unit_list, max_value_count, is_oem, support_search_value}` | OFFICIAL | **`PRIMARY_VERIFIED`** (SPD-023) — schema read Phase 02.3 |
| Requiredness | **static per-category boolean `mandatory`**. No conditional-requiredness mechanism exists (no analogue of ML's `POST .../attributes/conditional`). | OFFICIAL | `PRIMARY_VERIFIED` (SPD-023) |
| Free-text vs value-id | both, per `input_type` — `TEXT_FILLING` allows free text; dropdowns need a `value_id` | OFFICIAL | `SEARCH_INDEXED` |
| Units | `value_unit` on quantitative attributes | OFFICIAL | `SEARCH_INDEXED` |
| Regulatory fields | appear as **ordinary attributes** in regulated categories (INMETRO number, ANVISA registration, …) — not a separate subsystem | OFFICIAL | `SEARCH_INDEXED` |

## 3. Classification for the Skill

- **Attribute requiredness is `DYNAMIC`** — always fetch `get_attribute_tree` for
  the resolved leaf; never hardcode which attributes a category needs.
- **Mandatory (`mandatory = true`) attribute missing** → `PUBLICATION_STATUS =
  FAIL` after the category is resolved (`error_less_required_attribute`).
- **Recommended / "quality" attributes** → `QUALITY_STATUS`, never
  `PUBLICATION_STATUS` (same split the ML Skill makes for technical specs).
- **Check pending** (category or `get_attribute_tree` not yet resolved) →
  `PUBLICATION_STATUS = REVIEW`, `dynamic_checks_required:
  resolve_category_attributes`.

## 4. Open questions

- Conditional / shop-specific requiredness — **resolved**: no such mechanism;
  `mandatory` is a static per-category boolean (`PRIMARY_VERIFIED`, SPD-023).
- `input_type` = 1–5 (`SINGLE_DROP_DOWN` / `SINGLE_COMBO_BOX` / `FREE_TEXT_FILED`
  / `MULTI_DROP_DOWN` / `MULTI_COMBO_BOX`); `format_type` = NORMAL=1 /
  QUANTITATIVE_WITH_UNIT=2 (`PRIMARY_VERIFIED`, SPD-023). For a custom value,
  `add_item` takes `value_id = 0` + `original_value_name`.
- `support_search_value` → `v2.product.search_attribute_value_list` (page **not
  in the primary corpus** — `STILL_UNVERIFIED`).
- Whether regulated-category attributes (INMETRO / ANVISA numbers) are validated
  by Shopee or merely stored — `PRIMARY_PARTIAL` (regulated inputs surface as
  ordinary attributes; NCC/BSMI/FDA errors appear in `add_item`; INMETRO/ANVISA
  specifics not in corpus). See `references/compliance.md`.
- Brand is handled separately — `references/brand-and-identifiers.md`.

## 5. Dynamic-check placeholder

```
check: resolve_category_attributes
why:   determine which attributes are mandatory (field: `mandatory`) vs recommended
source: get_attribute_tree for the resolved leaf  (recommended set: get_recommend_attribute)
        ← endpoint NOT hardcoded; see api-and-auth.md §3 / §5
pending:                REVIEW (PUBLICATION)
executed + all mandatory filled: no blocker
executed + a mandatory attribute unmet: FAIL (PUBLICATION) — `error_less_required_attribute`
verification: PRIMARY_VERIFIED (SPD-023) — endpoint, `mandatory` field, input_type/format_type enums
```

## Sources

- **PRIMARY** — `docs/marketplaces/shopee/open-platform/product/attribute/get-attribute-tree.pdf`
  (`SPD-023`) + `get-recommend-attribute.pdf` (`SPD-024`) — Shopee Open Platform
  API Reference — read 2026-09-03 — `PRIMARY_VERIFIED`: endpoint + `category_id_list`
  (max 20) + `language`; requiredness field is **`mandatory`** (bool); full
  `attribute_info` schema (`input_type` 1–5, `format_type` 1–2, …); no
  conditional-requiredness mechanism. Registry: `research/shopee-primary-docs/`.
- Resource name `get_attribute_tree` (+ `get_recommend_attribute`,
  `search_attr_value`) — `github.com/QuoVadis86/shopee-sdk`,
  `github.com/congminh1254/shopee-sdk` (`docs/managers/product.md`) — community
  SDKs — consulted 2026-08-28 — `SEARCH_INDEXED` (superseded by SPD-023).
- Earlier field-name guess `is_mandatory` — `github.com/wjp-letgo/shopeego`,
  `github.com/teacat/shopeego` — community SDKs — consulted 2026-08-28 —
  **corrected to `mandatory` by SPD-023**.
- Regulatory fields as ordinary attributes — `help.shopee.com.br` art. 76226 —
  Central de Ajuda — consulted 2026-08-28 — `SEARCH_INDEXED`.
- Required-vs-quality split — `.claude/skills/mercado-livre-listing-best-practices/references/quality-audit.md`
  — internal — architectural reference only.
- Full detail: `research/shopee-listing-skill/discovery-report.md` §5.
