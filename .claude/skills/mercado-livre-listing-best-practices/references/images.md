# Images

last_reviewed: 2026-08-27
source_last_updated: 2026-03-24 (Trabalhar com imagens, per requester) — ⚠ verify
volatile: true

## OFFICIAL constraints (⚠ verify against the live "Trabalhar com imagens" page)

| Rule | Value |
|---|---|
| Accepted formats | JPG, JPEG, PNG |
| Upload resolution | 1200 × 1200 px; larger images are resized down to fit |
| Min resolution | 500 × 500 px (per requester's note) |
| Max resolution | 1920 × 1920 px (per requester's note) |
| Max file size | 10 MB per image |
| Color space | RGB preferred over CMYK |
| Product occupation | product should fill ~95% of the frame |
| Max images per item | **DYNAMIC** — read `settings.max_pictures_per_item` for the category |
| Max images per variation | **DYNAMIC** — read `settings.max_pictures_per_item_var` for the category |
| Main image | high quality; typically pure/white background, product only, no text/watermark/logos-overlay/borders/collage |
| Intellectual property | only images you have the right to use; no competitor images, no stock images you don't license |

Do **not** hardcode a maximum image count — it varies by category. Fetch the two
`max_pictures_per_item*` settings at listing time.

## INTERNAL — high-conversion gallery plan

Each image must do a commercial job. Don't add images just to hit a number.
Adapt the set to the product and category; this is **not** an ML requirement.

| # | Role | Purpose |
|---|---|---|
| 1 | **Hero** | Clean main shot, product fills frame, white background, correct variant. |
| 2 | Second angle | Back / side / 3-4 view to remove shape doubt. |
| 3 | Lifestyle / in context | Product in real use environment for scale and desire. |
| 4 | Primary benefit | Visualize the #1 reason to buy. |
| 5 | Close-up | Texture, finish, stitching, ports, controls. |
| 6 | Material / build | Show what it's made of; call out real materials. |
| 7 | Dimensions | Measured drawing / product next to a reference object. |
| 8 | Demonstration / how it works | Steps, assembly, operation. |
| 9 | What's in the box | Everything included laid out; note what is NOT included. |
| 10 | Variants | The available options, each labeled, matching the picker. |

Order and selection depend on the product. Skip roles that don't apply.

## Product identity fidelity (see also `return-prevention.md`)

AI-generated or AI-edited images must NOT change any essential characteristic:
shape, proportion, color, number of pieces, finish, stones, buttons, connectors,
ports, logos, included accessories, structure, or included packaging.

- Every image must depict the actual product sold.
- For a variant, the image must show **that** variant (color/size/finish).
- No borrowed photos, no rendered product that differs from the real unit.
- Background clean-up and lighting correction are fine; identity changes are not.

## Per-variant images

- Map each variant to its own image set (respecting `max_pictures_per_item_var`).
- The hero of each variant shows that variant.
- Do not reuse one color's photos for another color.

## Audit checks (`IMAGES`)

- [ ] Format / resolution / size / color space within OFFICIAL limits.
- [ ] Count ≤ category's DYNAMIC `max_pictures_per_item` / `_var`.
- [ ] Main image: product only, ~95% frame, no text/watermark/logo overlay.
- [ ] Every image = a real depiction of the real product; identity unaltered by any AI edit.
- [ ] Variant images match the correct variant.
- [ ] Each image has a defined role; no filler images.
- [ ] Rights to every image are clear (no competitor/stock misuse).

## Sources

- Trabalhar com imagens — https://developers.mercadolivre.com.br/pt_br/trabalhar-com-imagens — Developers — updated 2026-03-24 (per requester) ⚠ verify — consulted 2026-08-27 — formats, 1200×1200, 10 MB, RGB, 95%, `max_pictures_per_item(_var)`.
- Como criar anúncios eficientes — https://vendedores.mercadolivre.com.br/nota/como-criar-anuncios-eficientes-no-mercado-livre — Central de Vendedores — 2026 ⚠ verify — consulted 2026-08-27 — good photos as a listing-quality lever.
- Gallery roles table: INTERNAL (our operation), informed by general marketplace practice.
