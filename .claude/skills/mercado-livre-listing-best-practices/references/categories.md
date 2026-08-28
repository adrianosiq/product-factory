# Category & domain

last_reviewed: 2026-08-27
volatile: true

Everything category-specific is **DYNAMIC**. Never hardcode a category's
attributes, limits or allowed values as a global rule.

## 1. Predict the category (OFFICIAL ⚠ verify)

- Run the **category predictor** before publishing:
  `GET /sites/MLB/domain_discovery/search?q=<product name + key attrs>`.
- It returns candidate `category_id` + `domain_id` + predicted attributes.
- The agent must confirm the predicted category makes sense for the real product;
  if the predictor is low-confidence or wrong, escalate rather than force-fit.
- A wrong category cascades into wrong attributes, wrong filters and lost
  visibility — treat "category not confirmed via predictor" as a blocker.

## 2. Pull the attribute model (DYNAMIC)

- `GET /categories/$CATEGORY_ID/attributes` — the **static** attribute model for
  the category; for each attribute:
  - `id`, `name`, `value_type`, `values[]` (allowed `value_id` + `value_name`)
  - `tags`: `required`, `new_required` (mandatory when `condition = new`),
    `conditional_required`, `catalog_required`, `variation_attribute`,
    `allow_variations`, `hidden`, `read_only`, etc.
  - `PARENT_PK` / `CHILD_PK` markers for family/variation grouping
  - The `conditional_required` tag only marks an attribute as *possibly*
    required — it does not by itself say the attribute is required for a given
    item.
- `POST /categories/$CATEGORY_ID/attributes/conditional` — resolve which
  `conditional_required` attributes actually apply. The request body is the
  assembled item payload (category, price, condition, chosen attributes, …):
  the evaluation depends on item data, so this is **not** a static category
  lookup. (OFFICIAL — verified 2026-08-27)
- `GET /categories/$CATEGORY_ID` — `settings` block: `max_title_length`,
  `max_pictures_per_item`, `max_pictures_per_item_var`, `listing_allowed`,
  `immediate_payment`, variation flags, `catalog_domain`. These are the
  category's own limits (static per category, but only ML is authoritative —
  read them, never hardcode).

## 3. Attribute selection rules

| Rule | Tag |
|---|---|
| Fill every `required` and (if new) `new_required` attribute before anything else. | OFFICIAL ⚠ verify |
| Resolve `conditional_required` by calling `POST /categories/$CATEGORY_ID/attributes/conditional` with the assembled item payload; fill whatever it returns as required. | OFFICIAL (endpoint verified 2026-08-27) |
| Fill `recommended`/optional attributes whenever the ProductMaster supports them — they feed search filters. | OFFICIAL ⚠ verify |
| Send `value_id` (not free text) whenever the attribute has a `values[]` list. Free text only for genuinely open attributes. | OFFICIAL ⚠ verify |
| `PARENT_PK` attributes must be identical across a family and are sent by `value_id` to avoid language mismatch. | OFFICIAL ⚠ verify |
| `CHILD_PK` and custom attributes may vary within a family; they contribute id + name to family calc. | OFFICIAL ⚠ verify |
| Never invent a `value_id`. If the real value is not in the list, use the closest legitimate option or an open/custom value, and flag for review. | INTERNAL |
| Use "Não se aplica" / `value_name: "N/A"` only when the attribute genuinely does not apply — not to bypass a required field. | OFFICIAL ⚠ verify |

## 4. Variation axes → PARENT_PK / CHILD_PK

1. List the variation axes from the ProductMaster (color, size, voltage, …).
2. For each, check its attribute `tags` in the category model:
   - `allow_variations` / `variation_attribute` → it can be a variation axis.
   - `PARENT_PK` → it defines the family (same value across the family).
   - `CHILD_PK` → it varies within the family (the picker dimension).
3. If an axis the ProductMaster needs is not variable in this category, that's a
   category or catalog mismatch — flag it.

## 5. Domains

- The `domain_id` (from the predictor) governs how title / `family_name` and
  attributes are interpreted. Keep brand/model/line attributes consistent with
  the domain.
- Some domains have domain-level required attributes beyond the category — check.

## Sources

- Categorias e Atributos — https://developers.mercadolivre.com.br/pt_br/categorias-e-atributos-veiculos — Developers — ⚠ verify — consulted 2026-08-27 — attribute tags, `GET /categories/$ID/attributes`, PARENT_PK/CHILD_PK.
- Categories & attributes / What is an attribute? — https://developers.mercadolivre.com.br/en_us/categories-attributes , https://developers.mercadolivre.com.br/en_us/attributes — Developers — consulted 2026-08-27 (search-indexed copy; live page returns 403 to bots) — confirmed `POST /categories/$ID/attributes/conditional` takes the full item payload (title, category_id, price, currency_id, available_quantity, buying_mode, condition, listing_type_id, description, pictures, attributes, sale_terms) and returns which `conditional_required` attributes actually apply.
- Preditor de categorias — https://developers.mercadolivre.com.br/pt_br/categorizacao-de-produtos — Developers — ⚠ verify — consulted 2026-08-27 — predict category before publishing.
- Validações — https://developers.mercadolivre.com.br/pt_br/validacoes — Developers — ⚠ verify — consulted 2026-08-27 — `required` / `new_required` / `conditional_required` attribute tags.
