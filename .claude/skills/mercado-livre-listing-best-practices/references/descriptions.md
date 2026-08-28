# Description

last_reviewed: 2026-08-27
source_last_updated: 2026-03-13 (Descrição de produtos, per requester) — ⚠ verify

## OFFICIAL (⚠ verify against the live "Descrição de produtos" page)

- The description **complements** the ficha técnica. Fill the technical sheet
  first; use the description only for what the structured fields can't hold.
- API field is **`plain_text`** (JSON). Content must be plain text; if it isn't,
  the API returns a validation error indicating the character position that
  failed. No HTML.
- Organize the information; **avoid unnecessary redundancy** with the ficha técnica.
- Treat any character limit as **DYNAMIC** — confirm the current max rather than
  assuming one.

## INTERNAL — description structure

Use the sections that apply to the product; keep them short and scannable:

1. **Apresentação** — 1–2 sentences: what it is and who it's for.
2. **Benefícios** — the real, verifiable advantages (no superlatives without proof).
3. **Utilização** — how to use / install, in plain steps.
4. **Diferenciais** — what sets this product apart, factually.
5. **Especificações complementares** — only specs that don't fit a structured field.
6. **Conteúdo da embalagem** — exactly what's included; and a clear line for what
   is **NOT** included.
7. **Cuidados** — cleaning, storage, warnings.
8. **Compatibilidade** — summary pointer; authoritative data stays in the
   compatibility fields (`attributes.md`).
9. **Antes de comprar** — sizing/measurement notes, variant differences,
   illustrative-image disclaimer, anything that prevents a wrong expectation
   (feed from `return-prevention.md`).

## Avoid

- Keyword stuffing / synonym lists.
- Repeating the ficha técnica field-by-field.
- Unproven claims ("o melhor do mercado", "cura", "100% impermeável" without test).
- Invented features, non-existent warranties, fake certifications.
- Statements that contradict the attributes, images or `family_name`/title.
- Prices, external links, contact info, off-platform sales, promo countdowns.

Claims safety, prohibited/regulated products, brand/authenticity and
contact/external-diversion policy are the compliance layer — `compliance.md`.
An unsupported **removable** claim is dropped (`CONTENT_STATUS` may stay PASS);
an unsupported **essential** claim → REVIEW/FAIL by evidence.

## Audit checks (`DESCRIPTION`)

- [ ] `plain_text`, no HTML, passes format validation.
- [ ] Within the DYNAMIC character limit.
- [ ] No fact duplicated from the ficha técnica without reason.
- [ ] Every claim traces to CONFIRMED evidence.
- [ ] "What's included / not included" is explicit.
- [ ] "Antes de comprar" covers the return-prevention items relevant to this product.
- [ ] No contradictions with attributes / images / title / `family_name`.
- [ ] No prohibited content (links, contacts, off-platform, promo language).

## Sources

- Descrição de produtos — https://developers.mercadolivre.com.br/pt_br/descricao-de-produtos — Developers — updated 2026-03-13 (per requester) ⚠ verify — consulted 2026-08-27 — complements ficha técnica, `plain_text`, validation error with char position, avoid redundancy.
- Section structure and "avoid" list: INTERNAL, aligned with the OFFICIAL guidance above.
