# Attributes & ficha técnica

last_reviewed: 2026-08-27
volatile: true

## Core principle — structured first

Whenever a structured field exists for a fact, put the fact **there**, not only in
the description. Structured attributes are filterable, comparable and catalog-
matchable; description text is none of those. (OFFICIAL ⚠ verify — ficha técnica
comes before and feeds more than the description.)

## Fill order

1. `required` attributes (always).
2. `new_required` attributes (when `condition = new`).
3. `conditional_required` that actually apply — resolve via
   `POST /categories/$CATEGORY_ID/attributes/conditional`, sending the assembled
   item payload; fill whatever it returns as required. The `conditional_required`
   tag from `GET /categories/$CATEGORY_ID/attributes` only flags an attribute as
   *possibly* required; this POST call decides it for the specific item.
4. Variation attributes (`PARENT_PK` / `CHILD_PK`) — see `categories.md`.
5. `catalog_required` / `catalog_listing_required` if associating to catalog.
   Both are real attribute tags (confirmed 2026-08-28, search-indexed); the exact
   `catalog_required` vs `catalog_listing_required` distinction is still
   undocumented — treat each as "required for the applicable catalog operation,
   context-dependent" and do not invent a semantic difference.
6. **Technical-spec completeness** — `GET /categories/$CATEGORY_ID/technical_specs/input`.
   An attribute here whose requirement is **not** also carried by `required` in
   `GET .../attributes` improves completeness / search ranking only: missing it →
   the `incomplete_technical_specs` tag + a ranking penalty → `QUALITY_STATUS =
   REVIEW`, `PUBLICATION_STATUS` may still PASS. But the `technical_specs/input`
   response **can also surface `required` attributes** — decide publication-
   blocking from the resolved category attribute requirement model
   (`GET .../attributes` + the conditional check), not from where the attribute
   appeared. (OFFICIAL — verified 2026-08-27, search-indexed.)
7. All remaining attributes the ProductMaster supports (recommended + optional).

Requirement vs quality: `required` (in `.../attributes`) missing after category
resolution → `PUBLICATION_STATUS = FAIL`; `conditional_required` pending → REVIEW,
executed-and-unmet → FAIL; a technical-spec attribute that is *not* also
`required` → `QUALITY_STATUS = REVIEW` only. Certifications / regulatory data are
`CONDITIONAL_REQUIRED` whose *applicability* is a compliance question
(`compliance.md` §4) — never inferred from generic product type, never invented.

## Identity attributes — never invent

- **Product identifier (GTIN / EAN / UPC / …)**: see the dedicated section
  *"Product identifiers"* below — the requirement is CONDITIONAL_REQUIRED,
  legitimate absence uses `EMPTY_GTIN_REASON`, and an identifier is never
  invented.
- **Brand / Marca**: real brand only. If truly unbranded, use the "Sem marca" /
  "Genérico" value the category offers — do not put a seller/store name.
- **Model / Modelo / MPN**: manufacturer's, exactly. Keep identical to the title.
- **Line / Linha, Version / Versão**: only if confirmed.

## Value rules

- Send `value_id` when the attribute has an allowed `values[]` list.
- Match units to the attribute's expected unit (`value_struct` number + unit).
- If the real value isn't in the list: pick the closest legitimate option or use
  a permitted open/custom value, and raise a WARNING for human review — do not
  force an unrelated `value_id`.
- Do not leave a supported attribute blank just because it's optional.

## Product identifiers (GTIN & equivalents)

The identifier requirement is **CONDITIONAL_REQUIRED** (`SKILL.md` §2 B) — never
"every product needs a GTIN", never "GTIN is always optional". Resolve it from
the category model at listing time: `GET /categories/$CATEGORY_ID/attributes`
(is the identifier attribute — e.g. `GTIN` — tagged `required` / `new_required` /
`conditional_required`?) and, when `conditional_required`,
`POST /categories/$CATEGORY_ID/attributes/conditional` with the item payload
(see `categories.md`).

Identifier types ML accepts (OFFICIAL — verified 2026-08-27, search-indexed copy;
live page 403 to bots): **UPC** (GTIN-12, 12 digits), **EAN** (GTIN-13, 13
digits), **JAN** (8 or 13 digits), **ISBN** (books, 13 digits — convert ISBN-10
→ ISBN-13), **ITF-14** (multi-pack, 14 digits); auto parts also use **Part
Number**. Send the real code in the identifier attribute the category model
exposes. ML validates the identifier **format** on write and rejects a malformed
code — passing that format check is not evidence the code is this product's.

### Identifier state (procedural — not a stored enum)

| State | Meaning | Status effect |
|---|---|---|
| `KNOWN` | A real identifier supplied **with credible provenance** (GS1 / manufacturer / spec sheet). | ok |
| `CONDITIONAL_PENDING` | Whether an identifier is required is not resolved yet — category/API context pending. | REVIEW (pending dynamic check) |
| `LEGITIMATELY_ABSENT` | The product genuinely has no manufacturer identifier **and** ML's official absence mechanism for this category is satisfied (`EMPTY_GTIN_REASON`, below). | ok |
| `REQUIRED_MISSING` | Category/API confirms an identifier is mandatory, no valid identifier exists, and no accepted absence reason applies. | BLOCKER → FAIL |

### Evidence for a supplied identifier

- `KNOWN` needs provenance. A syntactically valid GTIN is **not** proof it
  belongs to this product.
- `UNSUPPORTED` — a code is present but its provenance does not tie it to this
  product → do not use it, do not publish it.
- `CONFLICTING` — two credible sources give different codes for the same physical
  product → stop, human review; never auto-pick, never publish.
- `MISSING` — no identifier information at all. **Not** the same as
  `LEGITIMATELY_ABSENT`: `MISSING` = "we don't know"; `LEGITIMATELY_ABSENT` =
  "confirmed none **and** official absence reason recorded".

### Never invent an identifier — BLOCKER-level integrity rule

Never generate, guess, infer from the product name, copy a competitor's code
without evidence it is the same product, or fabricate a code to satisfy a
mandatory attribute. An invented identifier is a BLOCKER.

### Legitimate absence — `EMPTY_GTIN_REASON` (OFFICIAL ⚠ verify)

When a product legitimately has no GTIN, ML's mechanism is the
**`EMPTY_GTIN_REASON`** attribute — not a literal `"N/A"` in the identifier
field.

- It is `conditional_required`: accepted only when the brand/domain has no GTIN
  loaded, and it can flip to **required** (reported: once a brand already has
  ~30 GTINs published — ⚠ verify the exact threshold). Resolve it through the
  conditional endpoint like any other `conditional_required` attribute.
- **Allowed values are category/domain-specific** — read them from
  `GET /categories/$CATEGORY_ID/attributes` for the `EMPTY_GTIN_REASON`
  attribute (`values[]`) and send a `value_id`. Do **not** hardcode a reason
  list (an example value ML documents is "Artisanal").
- "Identifier field absent" and "official absence reason resolved" are different
  states — do not substitute literal strings (`N/A`, `none`, `sem GTIN`, …) for
  the mechanism.

### Multi-variant products

An identifier belongs to the actual sellable unit. When variants each carry
their own manufacturer code, the listing has **one identifier per variant** (on
the variation / sale condition), not one code shared across the whole product.
Detailed variation-identifier mechanics are out of scope here — see
`references/variations-and-user-products.md`. (INTERNAL principle; ⚠ verify the
exact ML variation-level identifier fields.)

## Compatibility attributes (auto parts, accessories, electronics) — OFFICIAL ⚠ verify

- Use the dedicated compatibility mechanism, not the title/description:
  `POST /items/{MLB}/compatibilities`.
- Three modes: **universal** (fits all), **by catalog** (specific vehicle from
  ML's catalog), **by attributes** (brand + model + year + …).
- By-catalog / by-attributes require **at least 3 vehicle data points**
  (e.g. brand, model, year).
- For categories flagged `incomplete_compatibilities` (e.g. MLB22693 and the
  equivalent MLA/MLM auto-parts categories), declaring compatibility is
  **mandatory**.
- The title may still carry a short "compatível com X" only as a search aid — the
  authoritative data is the compatibility record.

## Evidence classification (run at step 1 of the workflow)

Tag every ProductMaster fact before using it:

| Tag | Meaning | Allowed in listing as fact? |
|---|---|---|
| **CONFIRMED** | Directly supported (spec sheet, official source, GS1 record, manufacturer). | Yes |
| **INFERRED** | Reasonably derived from confirmed data. | Only if low-risk; label internally |
| **MISSING** | No data. | No — add to `missing_information` |
| **CONFLICTING** | Sources disagree. | No — stop, human review (potential BLOCKER) |
| **UNSUPPORTED** | Claimed but no backing. | No — drop it |

Only CONFIRMED (and vetted INFERRED) facts may appear as attribute values or
description statements.

Evidence class answers *"can this value be used?"*; the requirement layer
(`SKILL.md` §2) answers *"does a gap here block?"*. They are independent: a
CORE_REQUIRED field that is MISSING blocks content creation; a COMMERCIAL_OPTIONAL
field that is MISSING only warns; either way a value with no backing is
UNSUPPORTED and is dropped, never guessed.

For product identifiers, `MISSING` ("we don't know") and `LEGITIMATELY_ABSENT`
("confirmed none + `EMPTY_GTIN_REASON` recorded") are distinct outcomes — see
*"Product identifiers"* below.

## Audit checks (`ATTRIBUTES`)

- [ ] All required / new_required filled; applicable conditional_required resolved via `POST .../attributes/conditional` and filled.
- [ ] `value_id` used wherever a list exists; units correct.
- [ ] Product identifier: `KNOWN` (provenance-backed), `LEGITIMATELY_ABSENT` via an `EMPTY_GTIN_REASON` `value_id`, or requirement `CONDITIONAL_PENDING`; never invented, never a literal `"N/A"` string. Brand/model real and consistent.
- [ ] No invented values; unresolved gaps are WARNINGs, not guesses.
- [ ] Compatibility declared via the dedicated mechanism where relevant / mandatory.
- [ ] Every value traces to CONFIRMED (or vetted INFERRED) evidence.

## Sources

- Categories & attributes (generic) — https://developers.mercadolivre.com.br/pt_br/categorias-e-atributos , https://developers.mercadolivre.com.br/en_us/attributes — Developers — SEARCH_INDEXED 2026-08-27 — attribute model, tags (`required` / `new_required` / `conditional_required` / `catalog_required` / `catalog_listing_required`), `technical_specs/input`. (The `…/categorias-e-atributos-veiculos` slug is the vehicles vertical only — use it for auto-parts compatibility, not generic rules.)
- Identificadores de produtos / Product identifiers — https://developers.mercadolivre.com.br/pt_br/identificadores-de-produtos , https://developers.mercadolivre.com.br/en_us/product-identifiers/ — Developers — verified 2026-08-27 (search-indexed copy; live page 403 to bots) — accepted identifier types (UPC/EAN/JAN/ISBN/ITF-14, Part Number), ISBN-10→13 and UPC-E→UPC-A conversion, GTIN format validated on write, "prioritise GTIN; otherwise send `EMPTY_GTIN_REASON`"; `EMPTY_GTIN_REASON` is `conditional_required`, allowed only when the brand has no GTIN loaded, can flip to required (~30-GTIN threshold ⚠ verify), values read from the category attribute model (example value "Artisanal").
- Categories & attributes / What is an attribute? — https://developers.mercadolivre.com.br/en_us/categories-attributes , https://developers.mercadolivre.com.br/en_us/attributes — Developers — consulted 2026-08-27 (search-indexed copy; live page returns 403 to bots) — `POST /categories/$ID/attributes/conditional` takes the full item payload and resolves which `conditional_required` attributes apply; `conditional_required` in `GET .../attributes` is only a "possibly required" flag.
- Códigos universais — https://www.mercadolivre.com.br/codigos-universais — Mercado Livre — ⚠ verify — consulted 2026-08-27.
- Compatibilidades de autopeças — https://developers.mercadolivre.com.br/pt_br/compatibilidades-itens-e-produtos-de-autopecas — Developers — ⚠ verify — consulted 2026-08-27 — 3 modes, ≥3 data points, mandatory categories, `/items/{MLB}/compatibilities`.
- O status das suas fichas técnicas — https://vendedores.mercadolivre.com.br/notas/o-status-das-suas-fichas-tecnicas/ — Central de Vendedores — 2026 ⚠ verify — consulted 2026-08-27 — completeness vs relevance.
