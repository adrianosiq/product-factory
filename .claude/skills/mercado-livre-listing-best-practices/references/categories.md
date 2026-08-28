# Category & domain

last_reviewed: 2026-08-27
volatile: true

Everything category-specific is **DYNAMIC**. Never hardcode a category's
attributes, limits or allowed values as a global rule.

## 1. Category resolution — discover, then validate

A category must end up **resolved** and **validated**. Neither means "returned by
the predictor". Track three procedural states (no new ProductMaster schema —
these are workflow states):

| State | Meaning |
|---|---|
| `category_candidate` | A category id has been *suggested/discovered* but not yet checked against the real product or current ML data. |
| `category_resolved` | One candidate has been selected and is supported by current product + marketplace context. |
| `category_validated` | The resolved category has been checked (leaf, listing-allowed, domain + attribute set fit the real product) sufficiently for the current workflow stage. |

### 1.1 Discovery — find candidates (use any; none is individually mandatory)

- **Category / domain predictor** — DISCOVERY tool, not proof of correctness
  (OFFICIAL — endpoint shape verified 2026-08-27, "discovery only" framing still
  ⚠ verify): `GET /sites/MLB/domain_discovery/search?q=<title / product + key
  attrs>` returns a **ranked list** of predictions (`domain_id`,
  `category_id`, `category_name`, predicted attributes); default 4, max 8, first
  = highest probability. ML frames it as a recommendation to help pick a
  category; it is not the sole source of truth and is not a required call when a
  candidate is already established by another current source.
- **Catalog association** — a matched `catalog_product_id` carries its category
  (`references/catalog.md`).
- **An existing ML listing/product** relationship for the same product.
- **A known taxonomy / internal category mapping** — usable as a candidate
  **only after** revalidation against current ML category data; a cached mapping
  can be stale and is never authoritative forever.

### 1.2 Validation — is this category usable and right for the product (OFFICIAL ⚠ verify)

Run against the selected candidate before it counts as `category_validated`:

- `GET /categories/$CATEGORY_ID` — the category must be a **leaf**
  (`children_categories` empty) and have `settings.listing_allowed = true`,
  otherwise it cannot host a listing. Also read `path_from_root` and `settings`
  (incl. `catalog_domain`).
- The `domain_id` and the attribute set (§2) must fit what the product actually
  is — a predicted-but-wrong category surfaces here as attributes that do not
  match the product.
- Cross-check against catalog context and the product's real characteristics.
- Low-confidence discovery, candidates that disagree, or a failed validation →
  **escalate**, do not force-fit.

A wrong category cascades into wrong attributes, wrong filters and lost
visibility. **No `category_validated` where the workflow needs one → BLOCKER.**
A candidate that is simply not yet validated because ML data is still pending →
REVIEW (Correction 02A: pending dynamic check, not `FAIL`).

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
| Use "Não se aplica" / `value_name: "N/A"` only when the attribute genuinely does not apply — not to bypass a required field. Product identifiers are the exception: legitimate absence of a GTIN uses the `EMPTY_GTIN_REASON` attribute (`attributes.md`), never a literal `"N/A"` in the identifier field. | OFFICIAL ⚠ verify |

## 4. Variation axes → PARENT_PK / CHILD_PK

1. List the variation axes from the ProductMaster (color, size, voltage, …).
2. For each, check its attribute `tags` in the category model:
   - `allow_variations` / `variation_attribute` → it can be a variation axis.
   - `PARENT_PK` → it defines the family (same value across the family).
   - `CHILD_PK` → it varies within the family (the picker dimension).
3. If an axis the ProductMaster needs is not variable in this category, that's a
   category or catalog mismatch — flag it.

## 5. Domains

- The `domain_id` (from discovery / carried by the resolved category) governs how
  title / `family_name` and attributes are interpreted. Keep brand/model/line
  attributes consistent with the domain.
- Some domains have domain-level required attributes beyond the category — check.

## Sources

- Categories & attributes (generic) — https://developers.mercadolivre.com.br/pt_br/categorias-e-atributos , https://developers.mercadolivre.com.br/en_us/categories-attributes — Developers — SEARCH_INDEXED 2026-08-27 — attribute tags, `GET /categories/$ID/attributes`, PARENT_PK/CHILD_PK. (The `…-veiculos` slug is the vehicles vertical only.)
- Categories & attributes / What is an attribute? — https://developers.mercadolivre.com.br/en_us/categories-attributes , https://developers.mercadolivre.com.br/en_us/attributes — Developers — consulted 2026-08-27 (search-indexed copy; live page returns 403 to bots) — confirmed `POST /categories/$ID/attributes/conditional` takes the full item payload (title, category_id, price, currency_id, available_quantity, buying_mode, condition, listing_type_id, description, pictures, attributes, sale_terms) and returns which `conditional_required` attributes actually apply.
- Preditor de categorias / Category prediction — https://developers.mercadolivre.com.br/pt_br/categorizacao-de-produtos , https://developers.mercadolivre.com.br/en_us/set-categories-for-products — Developers — consulted 2026-08-27 (search-indexed copy; live page 403 to bots) — `GET /sites/$SITE/domain_discovery/search?q=` returns a ranked list of predictions (domain_id, category_id, category_name, attributes), default 4 / max 8, first = highest probability; framed as a recommendation to pick a category, not authoritative validation.
- Set categories for your products — https://developers.mercadolivre.com.br/en_us/set-categories-for-products — Developers — consulted 2026-08-27 (search-indexed copy; live 403) — a listing must be created in a **leaf** category (`children_categories` empty) with `settings.listing_allowed = true`; posting in a non-leaf category is rejected.
- Validações — https://developers.mercadolivre.com.br/pt_br/validacoes — Developers — ⚠ verify — consulted 2026-08-27 — `required` / `new_required` / `conditional_required` attribute tags.
