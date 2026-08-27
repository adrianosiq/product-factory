# Variations & User Products

last_reviewed: 2026-08-27
source_last_updated: 2026-06-17 (User Products) / 2026 (Preço por variação) — ⚠ verify
volatile: true — highest-churn area of this Skill

## Step 0 — detect the model (DYNAMIC)

Confirm whether the seller/category operates on:

- **Traditional model** — one Item with `variations[]`, or
- **User Products / price-per-variation** — variants as independent offers under a
  User Product, grouped into a Family.

Do not assume. A seller migrated to User Products cannot be modeled with the old
`variations[]` mental model.

## Traditional model — `variations[]` (OFFICIAL ⚠ verify)

- Parent Item holds title, price, description, base pictures.
- Each entry in `variations[]`:
  - `attribute_combinations` — the variation axis values (e.g. Cor = Azul, Tamanho = M)
  - `available_quantity` — per-variant stock
  - `price` — generally inherits the parent (no true per-variant pricing)
  - `seller_custom_field` — SKU
  - `picture_ids` — per-variant images
- Only attributes marked as variation attributes for the category may be axes.

## User Products model (OFFICIAL ⚠ verify)

Objects: **Item / sale condition** → **User Product** → **Family**.

- **User Product** (`user_product_id`) — the seller's product record; holds the
  shared attributes and `family_name`.
- **Sale conditions / items** — independent offers under a User Product, each with
  its **own price, shipping and payment terms**, and its **own stock**. Reported
  cap: **max 30 per User Product** (⚠ verify exact number).
- **Family** (`family_id`, `family_name`) — groups User Products that share the
  same `PARENT_PK` values and differ on the picker (`CHILD_PK` / custom); shown as
  selectable pickers on the User Product Page.

### Grouping rules

| Attribute kind | Behavior |
|---|---|
| `PARENT_PK` | Must be **identical** across the whole family. Sent by `value_id`. Defines the family. |
| `CHILD_PK` | May **vary** within the family — this is the picker dimension. Contributes id + name to the family calc. |
| Custom attributes | May vary; contribute id + name only. |

### Naming

- `family_name` is the shared human name (see `titles-and-family-name.md`). It
  must not encode the picker value.
- `title` is accepted for backward compatibility and mapped to `family_name`;
  `family_name` wins if both are sent.
- **UPtin**: Mercado Livre generates `family_name` — don't craft it.

## Shared vs specific — mapping from ProductMaster

| Data | Where it lives |
|---|---|
| Brand, model, line, material (if constant), core specs | Shared attributes on the User Product / `PARENT_PK` |
| Color, size, voltage, flavor, pack size (the axes) | `CHILD_PK` / variation attributes |
| Per-variant stock | Per sale condition / per `variations[]` entry |
| Per-variant price, shipping, payment (new model only) | Per sale condition |
| Per-variant images | Per variant (`picture_ids` / per-variant set), respecting `max_pictures_per_item_var` |
| SKU | `seller_custom_field` per variant |

## Migration cautions

- Don't blindly port old-model habits: per-variant pricing, independent shipping
  and the 30-condition cap only exist in the new model.
- Don't put the differentiator only in the title/description — it must be a real
  `CHILD_PK` attribute so the picker works and search filters see it.
- Re-check family grouping: wrong `PARENT_PK` values silently split or merge families.

## Audit checks (`VARIANTS`)

- [ ] Correct model detected and used.
- [ ] Axes are real category variation attributes; `PARENT_PK` constant, `CHILD_PK` varies.
- [ ] `family_name` shared and picker-agnostic.
- [ ] Per-variant stock present; per-variant price/shipping only where the model supports it.
- [ ] Sale conditions ≤ the DYNAMIC cap (≈30).
- [ ] Each variant has correct, variant-specific images.
- [ ] SKUs unique and mapped.

## Sources

- User Products — https://developers.mercadolivre.com.br/pt_br/user-products — Developers — 2026-06-17 (per requester) ⚠ verify — consulted 2026-08-27 — Family/UP objects, PARENT_PK/CHILD_PK grouping, `family_name`/`title` mapping, UPtin.
- Preço por variação — https://developers.mercadolivre.com.br/pt_br/preco-variacao — Developers — 2026 ⚠ verify — consulted 2026-08-27 — independent offers, per-variant price/shipping/stock, ~30-condition cap, rollout.
- Variations (tradicional) — https://developers.mercadolivre.com.br/pt_br/variacoes — Developers — ⚠ verify — consulted 2026-08-27 — `variations[]` structure.
