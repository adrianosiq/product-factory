# Brand & product identifiers (GTIN / EAN / UPC / ISBN)

last_reviewed: 2026-08-28
phase_02_3_reviewed: 2026-09-03
phase_02_2_reviewed: 2026-08-30
volatile: true
classification: OFFICIAL — brand object + No-Brand (`brand_id:0`) + GTIN `"00"` mechanism PRIMARY_VERIFIED (SPD-010/025/026/029); per-category requiredness DYNAMIC
phase_02_3_note: >-
  PRIMARY (SPD-025 get_brand_list, SPD-026 register_brand, SPD-010, SPD-029).
  `get_brand_list` is per **leaf category**; params `offset` / `page_size`
  (max 100) / `category_id` / `status` (1 normal, 2 pending) / `language`;
  returns `brand_list[{brand_id, original_brand_name, display_brand_name}]` +
  `is_mandatory` for the brand attribute. **`add_item` ALWAYS requires the
  `brand` object** (`brand_id` True + `original_brand_name` True — "No Brand if
  not brand"); **"No Brand" ⇒ `brand_id: 0`** (`error_invalid_brand` "Brand ID
  value should be '0'"). `register_brand` is a real API, **QC-reviewed**
  (`error_busi_pending_qc`), blacklist rules; `unsupport_region_for_register_brand`
  for some markets (BR support `PRIMARY_NOT_FOUND`). **GTIN — PRIMARY_VERIFIED**:
  `gtin_code` string; **`"00"` = "Item without GTIN"** (no `EMPTY_GTIN_REASON`
  analogue — the literal `"00"` IS the mechanism); validation from
  `get_item_limit.gtin_limit.gtin_validation_rule` ∈ `Mandatory` / `Flexible`
  (GTIN or "00") / `Optional`; **model-level** (item-level for default-model
  items); BR + TW local sellers.
phase_02_2_note: >-
  `get_brand_list` is a **per-`category_id`** resource — brand requiredness is
  category-linked, which supports modelling brand as CONDITIONAL_REQUIRED /
  PUBLICATION_REQUIRED, not CORE_REQUIRED. A `product/register_brand` endpoint
  name now appears in a community SDK (`SEARCH_INDEXED`; Phase 01 said "not
  confirmed"). See `research/shopee-api-contract/phase-02.2-report.md` §13.

## 1. Brand — what is known vs provisional

**Do NOT lock the Phase 01 conclusion "brand is universally mandatory in Shopee
Brasil."** What discovery actually supports:

> Phase 01 evidence indicates brand is a **material** Shopee Brasil listing
> attribute and **may be required in broad contexts**. The exact category / API
> requiredness is `⚠ verify`.

| Aspect | Finding | Tag | Verification |
|---|---|---|---|
| Brand is an attribute | yes; and in Shopee BR it is treated as a material attribute for the product | OFFICIAL | `SEARCH_INDEXED` |
| Brand list | `get_brand_list`, per category → `brand_id`, `original_brand_name`, `display_brand_name` | OFFICIAL | `SEARCH_INDEXED` |
| "No Brand" | API contract: **`brand_id: 0`** + `original_brand_name` "No Brand" (`error_invalid_brand` "Brand ID value should be '0'"). "Sem marca" is the localized BR Seller-Center **display label** for this same value, shown first in the list. | OFFICIAL | `PRIMARY_VERIFIED` (SPD-010/025) for the API form; "first option" framing `SEARCH_INDEXED` |
| Seller-registered brands | `register_brand` (`POST`) — QC-reviewed (`error_busi_pending_qc`); rejected custom brands fall back to the No-Brand value | OFFICIAL | `PRIMARY_VERIFIED` (SPD-026); auto-revert behaviour `SEARCH_INDEXED` |
| Rejection reasons | logo/name mismatch, spelling errors, wrong category, policy breaches | OFFICIAL | `SEARCH_INDEXED` |
| Brand-restricted categories | some categories require brand authorisation (IP) | OFFICIAL | `SEARCH_INDEXED`; specifics `UNVERIFIED` |
| Registration API | **`POST /api/v2/product/register_brand`** is real (SPD-026): `original_brand_name` ≤ 254, `category_list` (L1/L2, max 50), `brand_region`, images; **QC-reviewed** (`error_busi_pending_qc`); blacklist rules; `unsupport_region_for_register_brand` for some markets (**BR support `PRIMARY_NOT_FOUND`**) | OFFICIAL | `PRIMARY_VERIFIED` (SPD-026); BR availability `PRIMARY_NOT_FOUND` |

## 2. Brand — how the Skill treats it

- Brand belongs to **publication / category resolution**, not product identity —
  unless the product's identity genuinely depends on brand (e.g. a branded
  collectible). It is **not** automatically `CORE_REQUIRED` (SKILL.md §3).
- The **value** is evidence-gated: a real, evidence-backed brand → use it; a
  genuine unbranded / generic product → the **No Brand** value (`brand_id: 0` +
  `original_brand_name` "No Brand"; "Sem marca" in the BR UI). **Never invent a
  brand** to satisfy the field.
- Readiness:
  - brand requirement for the category not resolved → `PUBLICATION_STATUS =
    REVIEW`, `dynamic_checks_required: resolve_brand_requirement`.
  - resolved as required for the category **and** unset → `PUBLICATION_STATUS =
    FAIL`.
  - IP-gated category + no brand authorisation → `PUBLICATION_STATUS` /
    `EXECUTION_STATUS = FAIL` (`resolve_brand_authorisation`).
  - custom brand pending Shopee approval → `EXECUTION` / `QUALITY = REVIEW`;
    fallback is "Sem marca".
- A brand claimed on a competitor listing or a review is **not** evidence the
  product is that brand.

## 3. Product identifiers — safe principle

> **Never invent GTIN / EAN / UPC / ISBN / ITF-14.**

| Aspect | Finding | Tag | Verification |
|---|---|---|---|
| Requiredness | `DYNAMIC` — `get_item_limit.gtin_limit.gtin_validation_rule` ∈ `Mandatory` / `Flexible` / `Optional` per shop+category (BR + TW local sellers) | OFFICIAL / DYNAMIC | `PRIMARY_VERIFIED` source (SPD-029) |
| GS1 framing | EAN presented as best practice, not a universal Shopee rule | external | `SEARCH_INDEXED` |
| "No GTIN" mechanism | **`gtin_code = "00"`** ("Item without GTIN") — the literal sentinel **is** the mechanism; accepted only where the resolved rule is `Flexible` / `Optional`, never a blanket accept. No `EMPTY_GTIN_REASON` reason-code analogue. | OFFICIAL | `PRIMARY_VERIFIED` (SPD-010/029) |
| Level | **model-level** `gtin_code` (item-level for default-model items); GS1 8–14 digits | OFFICIAL | `PRIMARY_VERIFIED` (SPD-010/015/017/020) |

## 4. Identifiers — how the Skill treats them

- Treat as `CONDITIONAL_REQUIRED`, resolved per category via
  **`get_item_limit.gtin_limit.gtin_validation_rule`**
  (`resolve_identifier_requirement`).
- State per product: `KNOWN` (provenance-backed), `CONDITIONAL_PENDING`
  (rule not resolved → `REVIEW`), or `REQUIRED_MISSING` (rule = `Mandatory` +
  absent → `PUBLICATION_STATUS = FAIL`; also `CONTENT` if the copy asserts a
  code).
- An invented code, or a competitor's code with no same-product evidence →
  `BLOCKER`.
- Shopee's structured "no barcode" path is **`gtin_code = "00"`** — use it only
  where the resolved rule is `Flexible` or `Optional`. Do **not** build a
  separate `EMPTY_GTIN_REASON` reason-code enum — the `"00"` sentinel is the
  whole mechanism. Where the rule is `Mandatory` and the product genuinely has no
  barcode, that is a `REVIEW` for human resolution, never a fabricated value.

## Sources

- Brand mandatory attribute + "Sem marca" first option + seller registration /
  approval / auto-revert — `seller.shopee.com.br/edu` (art. 10619), BR
  integrators (`base.com`, `gobots`, `mambadigital`) — Centro de Educação /
  external — consulted 2026-08-28 — `SEARCH_INDEXED`.
- `get_brand_list` (per `category_id`, paged, `status` filter) + `register_brand`
  method name — `github.com/wjp-letgo/shopeego`, `github.com/QuoVadis86/shopee-sdk`,
  `github.com/congminh1254/shopee-sdk` — community SDKs — consulted 2026-08-28 —
  `SEARCH_INDEXED`; `phase-02.2-report.md` §13, §29 (C5).
- EAN "obrigatório para alguns produtos" — `suporte.anymarket.com.br`,
  `atendimento.ideris.com.br` — external — consulted 2026-08-28 —
  `SEARCH_INDEXED`.
- EAN as best practice, not a blanket requirement — `blog.gs1br.org/como-vender-na-shopee`
  — GS1 Brasil — consulted 2026-08-28 — `SEARCH_INDEXED`.
- Full detail: `research/shopee-listing-skill/discovery-report.md` §16, §"Fact Table",
  §29 (U10, U11).
