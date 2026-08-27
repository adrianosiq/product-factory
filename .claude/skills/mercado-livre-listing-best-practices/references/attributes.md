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
3. `conditional_required` that actually apply (resolve via the conditional endpoint).
4. Variation attributes (`PARENT_PK` / `CHILD_PK`) — see `categories.md`.
5. `catalog_required` if associating to catalog.
6. All remaining attributes the ProductMaster supports (recommended + optional).

## Identity attributes — never invent

- **GTIN / EAN / UPC / código universal**: use the real code from the
  ProductMaster. If the product genuinely has none (artisanal, not factory-made,
  bundle), set the "não se aplica" option ML provides — this is allowed and does
  not disadvantage the listing. Never fabricate a code, never reuse another
  product's code.
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

## Audit checks (`ATTRIBUTES`)

- [ ] All required / new_required / applicable conditional_required filled.
- [ ] `value_id` used wherever a list exists; units correct.
- [ ] GTIN real or legitimately "não se aplica"; brand/model real and consistent.
- [ ] No invented values; unresolved gaps are WARNINGs, not guesses.
- [ ] Compatibility declared via the dedicated mechanism where relevant / mandatory.
- [ ] Every value traces to CONFIRMED (or vetted INFERRED) evidence.

## Sources

- Categorias e Atributos — https://developers.mercadolivre.com.br/pt_br/categorias-e-atributos-veiculos — Developers — ⚠ verify — consulted 2026-08-27.
- Identificadores de produtos — https://developers.mercadolivre.com.br/pt_br/identificadores-de-produtos — Developers — ⚠ verify — consulted 2026-08-27 — accepted codes, "não se aplica".
- Códigos universais — https://www.mercadolivre.com.br/codigos-universais — Mercado Livre — ⚠ verify — consulted 2026-08-27.
- Compatibilidades de autopeças — https://developers.mercadolivre.com.br/pt_br/compatibilidades-itens-e-produtos-de-autopecas — Developers — ⚠ verify — consulted 2026-08-27 — 3 modes, ≥3 data points, mandatory categories, `/items/{MLB}/compatibilities`.
- O status das suas fichas técnicas — https://vendedores.mercadolivre.com.br/notas/o-status-das-suas-fichas-tecnicas/ — Central de Vendedores — 2026 ⚠ verify — consulted 2026-08-27 — completeness vs relevance.
