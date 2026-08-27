# Title & family_name

last_reviewed: 2026-08-27
volatile: true

## First: which flow are you in?

| Flow | What to optimize |
|---|---|
| **Title is provided** (traditional Item, and current UP flows that still accept a seller title) | Craft the title with the OFFICIAL rules below, within the category's DYNAMIC length limit. |
| **Title is generated** (User Products where ML builds the title from `family_name` + attributes + domain; UPtin where ML also builds `family_name`) | Do NOT craft a title. Optimize `family_name`, attributes, domain and product traits so the generated title is good. |

`title` sent in a UP flow is mapped to `family_name`; `family_name` wins if both
are present (see `product-structure.md`). Never assume a fixed 60-character limit
— read `settings.max_title_length` for the category (DYNAMIC).

## OFFICIAL title rules (⚠ verify against Central de Vendedores)

- Structure: **Product + Brand + Model + main specification/attribute**, with the
  most decisive terms in roughly the first 40 characters.
- Be clear and objective; describe the product, not the deal.
- Do **not** include: "promoção", "oferta", "frete grátis", "último disponível",
  "envío grátis", store name, phone numbers, URLs, or calls to action.
- No emojis, no decorative special characters, no ALL CAPS words, no slang, no
  excess punctuation.
- Don't repeat the same word; don't stuff synonyms.
- Include brand and model exactly as they appear in the attributes (consistency).
- Include compatibility only when it's a primary buying signal and there's room
  (real compatibility data still belongs in the compatibility fields — see
  `attributes.md`).

## INTERNAL title construction recipe

1. Start from the attributes you already resolved: `PRODUCT_TYPE`, `BRAND`,
   `MODEL`/`LINE`, top 1–2 decisive specs (capacity, size, power, material,
   color if it's a search term).
2. Order: `<Tipo> <Marca> <Modelo/Linha> <Spec1> <Spec2> [<Cor/Variante se relevante>]`.
3. Trim to the DYNAMIC limit without dropping brand or model.
4. Remove filler ("original", "de qualidade", "top", "pronta entrega") unless it
   is itself a category attribute value.
5. Verify every token is true and supported by evidence (no invented specs).

## family_name (OFFICIAL ⚠ verify + INTERNAL)

- `family_name` is the product's canonical human name; it groups User Products
  into a Family and drives ML's generated title.
- Keep it stable across all User Products in the family — it must not encode the
  `CHILD_PK` picker value (e.g. don't bake the specific color/size into it when
  color/size is the picker).
- Build it from the shared identity: `<Tipo> <Marca> <Modelo/Linha> <shared specs>`.
- Consistency: `family_name` ⇄ brand/model attributes ⇄ description ⇄ images.
- For **UPtin**, ML sets `family_name` — don't fight it; make the underlying
  attributes and domain correct instead.

## Audit checks (`FAMILY_NAME_TITLE`)

- [ ] Correct flow detected (crafted vs generated).
- [ ] Length within the category's DYNAMIC `max_title_length` (no hardcoded 60).
- [ ] Brand and model present and identical to attribute values.
- [ ] No prohibited words / emojis / promo language.
- [ ] No repeated words, no keyword stuffing.
- [ ] `family_name` does not encode the picker dimension.
- [ ] Every claim in the title/`family_name` is CONFIRMED evidence.

## Sources

- Como criar um título atrativo — https://vendedores.mercadolivre.com.br/nota/como-criar-um-titulo-atrativo — Central de Vendedores — 2026 ⚠ verify — consulted 2026-08-27 — structure, prohibited words, clarity.
- Como criar anúncios eficientes — https://vendedores.mercadolivre.com.br/nota/como-criar-anuncios-eficientes-no-mercado-livre — Central de Vendedores — 2026 ⚠ verify — consulted 2026-08-27 — clear objective titles.
- User Products — https://developers.mercadolivre.com.br/pt_br/user-products — Developers — 2026-06-17 (per requester) ⚠ verify — consulted 2026-08-27 — title→family_name mapping, generated titles, UPtin.
- External (strategy only): conectaads.com, universidademarketplaces.com.br — PMME ordering heuristic (INTERNAL).
