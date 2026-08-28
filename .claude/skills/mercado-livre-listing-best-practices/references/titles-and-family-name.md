# Title & family_name

last_reviewed: 2026-08-27
volatile: true

## Step 1 — detect the title mode before writing anything

| Mode | When | What you control |
|---|---|---|
| **MANUAL / TRADITIONAL** | Traditional Item, and UP flows that still accept a seller `title`. | You send/craft the `title`. |
| **GENERATED / USER PRODUCTS** | UP flows where the human-visible name is `family_name`; **UPtin**, where ML also creates `family_name`. | You optimise `family_name`, attributes and domain — **not** a crafted `title`. |
| **UNRESOLVED** | Seller/account/category title mode not yet known (marketplace/account context unavailable). | Record a dynamic check; do not craft an irreversible `title`; status `REVIEW` while pending (Correction 02A). |

- A `title` sent in a UP flow is mapped to `family_name`; `family_name` wins if
  both are present (`product-structure.md`).
- Do **not** blindly craft a traditional `title` before the mode is determined.
- Sending a manual `title` in a flow where the current API generates it, or
  violating a required generated-title / `family_name` mechanism, is a **BLOCKER**
  (mode resolved and incompatible). Mode *merely unresolved* → `REVIEW`, not
  `FAIL`.
- Whether/how ML further composes a *displayed* title from `family_name` +
  attributes + domain is not something we have verified — ⚠ verify; do not build
  strategy on an assumed composition.

## OFFICIAL title guidance — manual flow

(Central de Vendedores "Como fazer um bom título" — verified 2026-08-27 against a
search-indexed copy; live page 403 to bots, so keep `⚠ verify` until a live read.)

- Recommended structure: **produto + marca + modelo + especificações** que ajudem
  o cliente a identificar o produto. This is a *recommendation*, not a fixed
  schema: it does **not** make brand or model mandatory, does not fix the word
  order, and does not require a crafted title in generated-title flows. Include
  brand/model only when they legitimately exist (see *Brand and model* below).
- Separate words with spaces; **no punctuation or symbols**.
- **Do not include** information about other services: devoluções, frete grátis,
  parcelamento; nor promo words ("promoção", "oferta", "último disponível"); nor
  store name, phone numbers, URLs, calls to action.
- **Do not include colour or size** — they live in the variation / ficha and are
  shown elsewhere in the listing.
- **Do not include "usado" / "recondicionado"** — condition is shown separately.
- **Do not reference other brands** with "tipo", "similar a", "igual a".
- No emojis, no ALL CAPS, no repeated words, no synonym stuffing.

> **Not an OFFICIAL rule:** the "put the key terms in the first ~40 of 60
> characters" heuristic comes from third-party SEO material, not ML
> documentation. The character *limit* is DYNAMIC (`max_title_length`), not a
> fixed 60, and no 40-character threshold is documented. See INTERNAL /
> EXPERIMENTAL below.

## DYNAMIC — length

- **Manual title**: `settings.max_title_length` for the category
  (`GET /categories/$CATEGORY_ID`). Never assume 60.
- **`family_name`**: must be **≤ the domain's `max_title_length`** (OFFICIAL —
  User Products docs, ⚠ verify). Do not carry a manual-flow number to
  `family_name` or vice-versa.

## INTERNAL — manual title construction

1. Start from resolved attributes: `PRODUCT_TYPE`, and `BRAND` / `MODEL` / `LINE`
   *if they exist*, plus the top 1–2 decisive specs.
2. Lead with the most product-defining terms (type, then brand/model if present,
   then the decisive spec) so the title is scannable and the defining words are
   not lost when the field is truncated in list/search views. This is a
   readability / clarity choice, **not** a known ranking rule.
3. Order guide: `<Tipo> <Marca?> <Modelo/Linha?> <Spec1> <Spec2>` — omit any
   element that does not legitimately exist; do not add colour / size / condition.
4. Trim to the DYNAMIC limit; drop filler ("original", "de qualidade", "top",
   "pronta entrega") unless it is itself an attribute value.
5. Every token must be CONFIRMED evidence — no invented specs, brand or model.

## EXPERIMENTAL (test with performance data; never stated as fact)

- Whether front-loading the most product-defining terms (vs. placing them later)
  measurably improves discovery or CTR — this is the testable form of the old
  "first 40 characters" heuristic, with no magic number.
- Whether a specific token order or title pattern converts better in a given
  category.

## family_name

- `family_name` is the **shared, stable identity of the product family**. Every
  User Product in a family carries the **same** `family_name`; family members are
  shown as pickers. (OFFICIAL — User Products docs, ⚠ verify.)
- It must **not** encode a `CHILD_PK` picker value — do not bake the specific
  colour / size / voltage into it when that is the picker dimension. (Minimal
  note here; full `PARENT_PK` / `CHILD_PK` handling lives in
  `variations-and-user-products.md`.)
- Build it from the shared identity: `<Tipo> <Marca?> <Modelo/Linha?> <shared
  specs>` — same brand/model rules as titles (only if they legitimately exist and
  are evidence-backed).
- Length ≤ the domain's `max_title_length` (DYNAMIC).
- The seller / integrator manages `family_name`; for **UPtin**, ML creates it —
  make the underlying attributes and domain correct instead of fighting it.
  (OFFICIAL — User Products docs, ⚠ verify.)
- Consistency: `family_name` ⇄ brand/model attributes ⇄ description ⇄ images.

Illustrative only (not an ML format): family `Óculos Quadrado Unissex Acetato`;
the picker variants `Preto` / `Transparente` / `Tartaruga` live in the variation
attribute, not in `family_name`.

## Brand and model in titles / family_name

- Use brand / model / MPN **only when they legitimately exist** and are relevant
  or required. A genuine generic / unbranded product gets **no** fabricated
  brand — use the category's "Sem marca" / "Genérico" attribute value
  (`attributes.md`) and simply omit a brand from the title.
- When they exist: the value must be evidence-backed (CONFIRMED) and
  **consistent** across ProductMaster ⇄ structured attributes ⇄ title /
  `family_name` ⇄ description.
- Never invent a brand, model or MPN to satisfy a title template.

## Audit checks (`FAMILY_NAME_TITLE`)

- [ ] Title mode detected (manual / generated / unresolved); unresolved → dynamic
      check + `REVIEW`, not a crafted title.
- [ ] Manual title within the category's DYNAMIC `max_title_length`; `family_name`
      within the domain's `max_title_length`. No hardcoded 60.
- [ ] Brand / model / MPN present **only** where they legitimately exist,
      evidence-backed, consistent with attributes. None invented.
- [ ] No prohibited content: other-services info, promo words, store name / URL /
      phone, colour / size, "usado" / "recondicionado", other-brand references,
      emojis, ALL CAPS, punctuation / symbols.
- [ ] No repeated words, no synonym stuffing.
- [ ] `family_name` is the shared family identity and does not encode the picker
      dimension.
- [ ] Every term in the title / `family_name` traces to CONFIRMED evidence.
- [ ] No first-N-characters / ranking folklore relied on as if it were an ML rule.

## Sources

- Como fazer um bom título para o seu anúncio — https://vendedores.mercadolivre.com.br/nota/como-criar-um-titulo-atrativo — Central de Vendedores — verified 2026-08-27 (search-indexed copy; live page 403 to bots) — structure produto+marca+modelo+especificações; separate words with spaces, no punctuation/symbols; do not include other-services info, colour/size, condition, or other-brand references. **No 40-character threshold is stated.**
- Como criar anúncios eficientes / alcançar mais compradores — https://vendedores.mercadolivre.com.br/nota/como-criar-anuncios-eficientes-no-mercado-livre — Central de Vendedores — 2026 ⚠ verify — consulted 2026-08-27 — clear, objective titles as an "efficient listing" lever.
- User Products — https://developers.mercadolivre.com.br/pt_br/user-products — Developers — verified 2026-08-27 (search-indexed copy; live 403) — `family_name` ≤ the domain's `max_title_length`; the same `family_name` across a family (shown as pickers); seller/integrator manages `family_name`, ML creates it only for UPtin; `title` → `family_name` mapping.
- The "first ~40 characters" ordering heuristic and "first words weigh more in search" are INTERNAL / EXPERIMENTAL only — sourced to third-party SEO material (conectaads.com, universidademarketplaces.com.br, base.com), not ML documentation.
