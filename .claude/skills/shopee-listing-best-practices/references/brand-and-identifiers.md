# Brand & product identifiers (GTIN / EAN / UPC / ISBN)

last_reviewed: 2026-08-28
phase_02_3_reviewed: 2026-09-03
phase_02_2_reviewed: 2026-08-30
volatile: true
classification: OFFICIAL (existence of the requirement) — verification SEARCH_INDEXED; exact scope `⚠ verify`
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
| "No brand" | **"Sem marca"** is supported and is the **first option** in the list | OFFICIAL | `SEARCH_INDEXED` — treat as provisional / search-indexed until verified |
| Seller-registered brands | a seller can submit their own or the manufacturer's brand; **subject to Shopee approval**; listings auto-revert to "Sem marca" if rejected | OFFICIAL | `SEARCH_INDEXED` |
| Rejection reasons | logo/name mismatch, spelling errors, wrong category, policy breaches | OFFICIAL | `SEARCH_INDEXED` |
| Brand-restricted categories | some categories require brand authorisation (IP) | OFFICIAL | `SEARCH_INDEXED`; specifics `UNVERIFIED` |
| Registration API | a **`product/register_brand`** method name is now observed in community SDKs (Phase 02.2) — a brand-registration resource *appears* to exist; its **schema, approval behaviour and whether it is API- or Seller-Center-driven** are `UNVERIFIED` | — | `SEARCH_INDEXED` (name) / `UNVERIFIED` (behaviour) |

## 2. Brand — how the Skill treats it

- Brand belongs to **publication / category resolution**, not product identity —
  unless the product's identity genuinely depends on brand (e.g. a branded
  collectible). It is **not** automatically `CORE_REQUIRED` (SKILL.md §3).
- The **value** is evidence-gated: a real, evidence-backed brand → use it; a
  genuine unbranded / generic product → **"Sem marca"**. **Never invent a brand**
  to satisfy the field.
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
| Requiredness | an identifier **appears mandatory for some products / categories** ("obrigatório para alguns produtos") — **not** a blanket listing requirement | OFFICIAL / DYNAMIC | `SEARCH_INDEXED` |
| GS1 framing | EAN presented as best practice, not a universal Shopee rule | external | `SEARCH_INDEXED` |
| Structured absence | **no** confirmed "legitimately has no barcode" mechanism (no `EMPTY_GTIN_REASON` analogue) | — | `UNVERIFIED` |
| Level | item vs model level, duplicate rules across models | — | `UNVERIFIED` |

## 4. Identifiers — how the Skill treats them

- Treat as `CONDITIONAL_REQUIRED`, resolved per category via `get_attributes` /
  policy (`resolve_identifier_requirement`).
- State per product: `KNOWN` (provenance-backed), `CONDITIONAL_PENDING`
  (requiredness not resolved → `REVIEW`), or `REQUIRED_MISSING` (resolved
  required + absent → `PUBLICATION_STATUS = FAIL`; also `CONTENT` if the copy
  asserts a code).
- An invented code, or a competitor's code with no same-product evidence →
  `BLOCKER`.
- **Do not** build an `EMPTY_GTIN_REASON` analogue unless Shopee is confirmed to
  provide one. If a genuine no-barcode product hits a category that requires an
  identifier, that is a `REVIEW` for human resolution, not a fabricated value.

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
