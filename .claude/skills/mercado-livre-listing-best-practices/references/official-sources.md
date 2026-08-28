# Official sources

last_reviewed: 2026-08-27
fetch_method_note: >-
  developers.mercadolivre.com.br, vendedores.mercadolivre.com.br and
  www.mercadolivre.com.br/ajuda all return HTTP 403 to bots — **no page in this
  file has been read live.** Each row carries an explicit verification quality:
  LIVE (read directly — none yet); SEARCH_INDEXED (corroborated against
  search-engine copies of the official page, with a date); UNVERIFIED / "⚠ verify"
  (not corroborated — treat as provisional and confirm before relying on it).
  A SEARCH_INDEXED rule is not LIVE and may still be stale; re-check per SKILL.md §5.

## Priority order for sources

1. **Mercado Livre Developers Brasil** — `https://developers.mercadolivre.com.br/pt_br/...`
2. **Central de Vendedores / Central de Aprendizagem** — `https://vendedores.mercadolivre.com.br/...`
3. **Central de Ajuda Mercado Livre** — `https://www.mercadolivre.com.br/ajuda/...`
4. Other official Mercado Livre pages.

External blogs/tools may **only** inform commercial strategy (INTERNAL /
EXPERIMENTAL). They are never a source of OFFICIAL rules. Do not base an important
rule solely on LLM prior knowledge.

## Core official pages

| Page | URL | source_last_updated | Rules derived (see file) |
|---|---|---|---|
| Publicar produtos (guia) | https://developers.mercadolivre.com.br/pt_br/publicacao-de-produtos/ | ⚠ verify | product-structure, categories |
| User Products | https://developers.mercadolivre.com.br/pt_br/user-products , https://developers.mercadolibre.com.ar/en_us/user-products | 2026-06-17 (per requester); verified 2026-08-27 (search-indexed; live 403) — UP=physical product, `user_product_id` ML-assigned, 1 UP→many Items (sale conditions), `family_id`, `user_product_seller` / `user_product_listing` tags, post-activation unification, auto-generated title, async replication of product-level PUTs, family calc (PARENT_PK identical / CHILD_PK+custom id+name / `read_only` PK excluded), `GET /users/$SELLER_ID/items/search?user_product_id=`, `GET /sites/$SITE/user-products-families/$FAMILY_ID` | product-structure, variations-and-user-products, titles-and-family-name |
| Preço por variação / Price per variation | https://developers.mercadolivre.com.br/pt_br/preco-variacao | 2026; verified 2026-08-27 (search-indexed; live 403) — per-Item price/shipping/stock, ~30 sale conditions per UP, wave rollout, seller-request migration only | variations-and-user-products, pricing-and-commercial |
| Variations (legacy) | https://developers.mercadolivre.com.br/pt_br/variacoes | verified 2026-08-27 (search-indexed; live 403) — `variations[]` / `attribute_combinations` / `SELLER_SKU`; max 100 per item (250 Fashion / Mobile Accessories / Auto Parts), stated 2022-12-14 | variations-and-user-products |
| Estoque Multi Origem | https://developers.mercadolivre.com.br/pt_br/estoque-multi-origem | updated 2026-05-15; verified 2026-08-27 (search-indexed; live 403) — **MLB supported**, activation DYNAMIC; `warehouse_management` = single warehouse, `+ multiwarehouse` = multiple; `POST /items/multiwarehouse` + `stock_locations`; `available_quantity` not used once Multi Origem active; MLB warehouses must be in the same state as the seller's CNPJ | variations-and-user-products, product-structure |
| Estoque Distribuído | https://developers.mercadolivre.com.br/pt_br/estoque-distribuido | updated 2026-04-22; verified 2026-08-27 (search-indexed; live 403) — without Multi Origem, `PUT /items` `available_quantity` valid (ML syncs across a UP's Items); `selling_address` stock write (`PUT /user-products/{id}/stock/type/selling_address`) **MLA/MLC only**; `GET/PUT /user-products/{id}/stock[/type/seller_warehouse]`; `GET /users/{id}/stores/search?tags=stock_location`; `meli_facility` not seller-writable | variations-and-user-products, product-structure |
| Gestão de estoque multiorigem / User Products FAQ | https://developers.mercadolibre.com.ar/stock-multiwarehouse | updated 2026-08-14; verified 2026-08-27 (search-indexed; live 403) — sites where `selling_address` modification is blocked (incl. **MLB**) use the `seller_warehouse` flow; UP holds up to two typologies `(selling_address + meli_facility)` or `(seller_warehouse + meli_facility)` | variations-and-user-products |
| SELLER_SKU vs seller_custom_field | https://developers.mercadolibre.com.ar/en_us/variations | verified 2026-08-27 (search-indexed; live 403) — `SELLER_SKU` is the ML-recognised SKU attribute; `seller_custom_field` is seller-internal, unrelated | variations-and-user-products |
| Categories & attributes (generic) | https://developers.mercadolivre.com.br/pt_br/categorias-e-atributos , https://developers.mercadolivre.com.br/en_us/categories-attributes | SEARCH_INDEXED 2026-08-27 — generic attribute model, tags (`required` / `new_required` / `conditional_required` / `catalog_required` / `catalog_listing_required`), `PARENT_PK` / `CHILD_PK`, `technical_specs/input` | categories, attributes |
| Categorias e Atributos — Veículos (vehicles vertical only) | https://developers.mercadolivre.com.br/pt_br/categorias-e-atributos-veiculos | ⚠ verify | attributes (auto-parts compatibility) |
| Preditor de categorias / Category prediction | https://developers.mercadolivre.com.br/pt_br/categorizacao-de-produtos | verified 2026-08-27 (search-indexed; live 403) | categories |
| Set categories for your products (leaf-category rule) | https://developers.mercadolivre.com.br/en_us/set-categories-for-products | verified 2026-08-27 (search-indexed; live 403) | categories |
| Validações (fluxo de validação; tags de atributo) | https://developers.mercadolivre.com.br/pt_br/validacoes | ⚠ verify | categories, quality-audit |
| Validador de publicações (`POST /items/validate`) | https://developers.mercadolivre.com.br/pt_br/validador-de-publicacoes | verified 2026-08-27 (search-indexed; live 403) | quality-audit, SKILL.md §5 |
| Categories & attributes / What is an attribute? (`POST .../attributes/conditional`) | https://developers.mercadolivre.com.br/en_us/categories-attributes | verified 2026-08-27 (search-indexed; live 403) | categories, attributes, SKILL.md §5 |
| Identificadores de produtos / Product identifiers (GTIN, `EMPTY_GTIN_REASON`) | https://developers.mercadolivre.com.br/pt_br/identificadores-de-produtos , https://developers.mercadolivre.com.br/en_us/product-identifiers/ | verified 2026-08-27 (search-indexed; live 403) | attributes |
| Trabalhar com imagens / Working with pictures | https://developers.mercadolivre.com.br/pt_br/trabalhar-com-imagens , https://developers.mercadolivre.com.br/en_us/working-with-pictures | 2026-03-24 (per requester); verified 2026-08-27 (search-indexed; live 403) — formats, RGB, ~95%, 1200×1200 rec / 500×500 min / 1920×1920 max, ≤10 MB, zoom >~800 px, smartcrop, `POST /pictures/items/upload` | images |
| Fotos de qualidade… / Como tirar boas fotos dos seus produtos | https://vendedores.mercadolivre.com.br/nota/fotos-de-qualidade-o-segredo-para-se-destacar-e-vender-mais , https://www.mercadolivre.com.br/ajuda/Como-tirar-boas-fotos-dos-seus-produtos_1320 | 2026 ⚠ verify (search-indexed 2026-08-27) — cover-photo: product-only, prohibited overlays (logo/watermark/promo/QR/contact), category-dependent white-background requirement | images |
| Image moderation / Manage moderations / Image Diagnostics | https://developers.mercadolibre.com.ar/en_us/manage-moderations , https://developers.mercadolibre.com.ar/en_us/image-diagnostics | verified 2026-08-27 (search-indexed; live 403) — image-quality moderation, `poor_quality_thumbnail` on `active`/`paused`; Image Diagnostics (base64 / `picture_id` / URL; prioritise thumbnail) | images |
| Descrição de produtos | https://developers.mercadolivre.com.br/pt_br/descricao-de-produtos | 2026-03-13 (per requester) ⚠ verify | descriptions |
| Compatibilidades de autopeças | https://developers.mercadolivre.com.br/pt_br/compatibilidades-itens-e-produtos-de-autopecas | ⚠ verify | attributes (compatibility) |
| Informe compatibilidades | https://developers.mercadolivre.com.br/informe-compatibilidades | ⚠ verify | attributes (compatibility) |
| Sincronização e modificação de publicações | https://developers.mercadolivre.com.br/pt_br/produto-sincronizacao-de-publicacoes | ⚠ verify | product-structure |
| Como criar anúncios eficientes | https://vendedores.mercadolivre.com.br/nota/como-criar-anuncios-eficientes-no-mercado-livre | 2026 ⚠ verify | seo-and-discovery, titles-and-family-name, images |
| Como fazer um bom título para o seu anúncio | https://vendedores.mercadolivre.com.br/nota/como-criar-um-titulo-atrativo | verified 2026-08-27 (search-indexed; live 403) — structure produto+marca+modelo+especificações, spaces not punctuation, exclude other-services info / colour / size / condition / other-brand references; **no 40-character rule stated** | titles-and-family-name |
| O status das suas fichas técnicas | https://vendedores.mercadolivre.com.br/notas/o-status-das-suas-fichas-tecnicas/ | 2026 ⚠ verify | attributes, seo-and-discovery |
| Códigos universais | https://www.mercadolivre.com.br/codigos-universais | ⚠ verify | attributes |
| Qualidade das publicações / listing `/performance` | https://developers.mercadolivre.com.br/pt_br/qualidade-das-publicacoes | verified 2026-08-27 (search-indexed; live 403) — `/health` discontinued → `GET /item/$ITEM_ID/performance`; `level_wording`, entity `mode` OPPORTUNITY / WARNING | quality-audit, compliance |
| Technical specs input / `incomplete_technical_specs` | https://developers.mercadolivre.com.br/en_us/attributes (`GET /categories/$CATEGORY_ID/technical_specs/input`) | SEARCH_INDEXED 2026-08-27 — decide publication-blocking from the resolved category attribute requirement model (`required` in `.../attributes` + conditional check); `technical_specs/input` can *also* add completeness/exposure requirements beyond that hard set (tag `incomplete_technical_specs`, ranking only) — it is **not** wholesale quality-only | quality-audit, attributes |
| Manage moderations | https://developers.mercadolivre.com.br/en_us/manage-moderations | SEARCH_INDEXED 2026-08-28 — `GET /moderations/last_moderation/{MODERATION_REFERENCE_ID}` (`<element_id>-<element_type>`, suffix `-ITM` / `-QUE` / `-REV`); `GET /moderations/infractions/{USER_ID}` (query `related_item_id`, `element_id`, `element_type`, dates, pagination, sort). `reason` always / `remedy` only when recoverable. Infraction states `forbidden` / `waiting_for_patch` / `held` / `pending_documentation` — distinct from the Item's `active` / `paused` / `inactive`. Preventive pauses | compliance, quality-audit |
| Catalog required listings / `catalog_only_restricted` | https://developers.mercadolivre.com.br/pt_br/publicacoes-necessarias-do-catalogo , https://developers.mercadolivre.com.br/en_us/catalog-eligibility | verified 2026-08-27 (search-indexed; live 403) — catalog-exclusive → `under_review` + `catalog_only_restricted`; catalog-required → `catalog_listing_eligible` + `listing_strategy: catalog_required` → `opt_obey`; applies to MLB | compliance, catalog |
| Shipment handling / SELLER_PACKAGE_* (ME2) | https://developers.mercadolivre.com.br/en_us/shipment-handling | verified 2026-08-27 (search-indexed; live 403) — `SELLER_PACKAGE_HEIGHT/LENGTH/WIDTH/WEIGHT` mandatory for ME2 in some categories, not always in category attributes; cm + grams, real values; missing → moderated / not published | compliance, quality-audit |
| Produtos proibidos / Práticas proibidas / Propriedade intelectual | https://www.mercadolivre.com.br/ajuda/Produtos-proibidos_1029 , https://www.mercadolivre.com.br/ajuda/contornar_estrutura_programas_4822 , https://www.mercadolivre.com.br/ajuda/1078 , https://www.mercadolivre.com.br/ajuda/Propriedade-roubada_1034 | ⚠ verify (search-indexed 2026-08-27) — prohibited-product categories; non-compliance → removal + account penalties | compliance |
| Normas da ANVISA (regulated products) | https://vendedores.mercadolivre.com.br/nota/como-cumprir-as-normas-da-anvisa-e-evitar-o-cancelamento-do-seu-anuncio | ⚠ verify (search-indexed 2026-08-27) — regulated-product compliance example | compliance |
| Brand Protection Program | https://www.mercadolivre.com.br/ajuda/Programa-de-Prote%C3%A7ao-Propriedade-Intelectual_2099 , https://www.mercadolivre.com.br/brandprotection/enforcement | ⚠ verify (search-indexed 2026-08-27) — rights-holder report + seller response (4 calendar days; licence doc PDF/PNG/JPG ≤ 5 MB); no reply → auto-removal | compliance |
| Catálogo / Buy Box (Central) | https://vendedores.mercadolivre.com.br/ (buscar "catálogo") | ⚠ verify | catalog |

## API endpoints referenced by this Skill (DYNAMIC)

- `GET /sites/MLB/domain_discovery/search?q=...` — category / domain **discovery**
  (recommendation): ranked prediction list (`domain_id`, `category_id`,
  `category_name`, attributes), default 4 / max 8. Not authoritative validation;
  not a required call when a candidate is already established. Verified 2026-08-27.
- `GET /categories/$CATEGORY_ID` — category resource for **validation** + limits:
  must be a **leaf** (`children_categories` empty) with
  `settings.listing_allowed = true` to host a listing; also `path_from_root`,
  `settings.catalog_domain`, `settings.max_pictures_per_item(_var)`,
  `settings.max_title_length` (the manual-title limit), variation flags (read,
  never hardcode). Verified 2026-08-27. For User Products, `family_name` must be
  ≤ the **domain's** `max_title_length` — verified 2026-08-27 (search-indexed).
- `GET /categories/$CATEGORY_ID/attributes` — the **static** attribute model:
  attribute ids, `values`, `tags` (`required`, `new_required`,
  `conditional_required`, `catalog_required`), `PARENT_PK`, `CHILD_PK`. Also the
  source of the product-identifier attribute and of `EMPTY_GTIN_REASON`'s allowed
  `values[]` (never hardcode the reason list).
- `POST /categories/$CATEGORY_ID/attributes/conditional` — send the assembled
  item payload; returns which `conditional_required` attributes actually apply
  for that item (not a static lookup). Verified 2026-08-27.
- `GET /categories/$CATEGORY_ID/technical_specs/input` — technical-completeness
  attributes (search-ranking; `incomplete_technical_specs`). Mostly →
  `QUALITY_STATUS`, but the response can also surface attributes that are
  `required` — decide publication-blocking from the resolved category attribute
  requirement model, not from this endpoint alone. SEARCH_INDEXED 2026-08-27.
- `GET /item/$ITEM_ID/performance` — post-publication listing quality (score,
  `level_wording`, `OPPORTUNITY`/`WARNING` entities). **Replaces `/health`.**
  → `QUALITY_STATUS`. SEARCH_INDEXED 2026-08-27. (No separate
  `/user-product/{id}/performance` endpoint is confirmed — do not add one.)
- `GET /moderations/last_moderation/{MODERATION_REFERENCE_ID}` — last moderation
  for one element; reference = `<element_id>-<element_type>` (`-ITM` for a
  listing). `GET /moderations/infractions/{USER_ID}` — seller infraction history
  (query: `related_item_id`, `element_id`, `element_type`, dates, pagination,
  sort). `reason` (always) + `remedy` (only when recoverable). Infraction states
  `forbidden` / `waiting_for_patch` / `held` / `pending_documentation` — distinct
  from the Item's `active` / `paused` / `inactive`. → `EXECUTION_STATUS` /
  compliance. SEARCH_INDEXED 2026-08-28.
- `GET /products/search?site_id=MLB&q=...` (also `GET /marketplace/products/search`
  — `site_id` required) and `GET /products/$CATALOG_PRODUCT_ID` — catalog match. SEARCH_INDEXED 2026-08-27.
- `POST /items/validate` — pre-publish technical validation of a listing
  payload; HTTP 204 when no problems are found, HTTP 400 with a `cause[]` array
  of errors/warnings otherwise. Optional (meant for testing); passing it does
  not guarantee publication or content correctness. Verified 2026-08-27.
- `POST /items/{MLB}/compatibilities`
- `GET /items/{MLB}/description` / `POST` with `plain_text`
- `POST /pictures/items/upload` — multipart upload of item images to ML's CDN
  (≤ 10 MB per file). Verified 2026-08-27 (search-indexed).
- Image Diagnostics API — validate an image (base64 / `picture_id` / URL) before
  use; prioritise the main image (thumbnail). Image moderation flags a listing
  `active`/`paused` with `poor_quality_thumbnail`. Future execution mechanism —
  MCP wiring out of scope. Verified 2026-08-27 (search-indexed).

**User Products & multi-origin stock** (verified 2026-08-27, search-indexed; live
403. **MLB Multi Origem is officially supported**; per-seller activation is
DYNAMIC — read the tags):

- `GET /users/$USER_ID` — seller tags: `user_product_seller` (new publication
  model active); `warehouse_management` (Multi Origem experience — **single**
  warehouse on its own); `warehouse_management` + `multiwarehouse` (**multiple**
  warehouses).
- `GET /user-products/$USER_PRODUCT_ID` — UP product-level data (`family_id`,
  attributes, identity). `GET /sites/$SITE/user-products-families/$FAMILY_ID` —
  all UPs in a family.
- `GET /users/$SELLER_ID/items/search?user_product_id=…` — Items for a UP
  (comma-separated list accepted).
- `PUT /items` `available_quantity` — the stock-write mechanism in **`STANDARD`**
  inventory mode (no `warehouse_management`), including User Products without
  Multi Origem; ML syncs across a UP's Items. **Not** used once Multi Origem is
  resolved-active.
- `POST /items/multiwarehouse` — create an Item with `stock_locations`
  (`store_id`, `network_node_id`, `quantity`) for a `warehouse_management` seller;
  `available_quantity` invalid; response returns `user_product_id` to persist.
- `GET /user-products/$USER_PRODUCT_ID/stock` — per-location `type`,
  `network_node_id`, `store_id`, `quantity`.
- `PUT /user-products/$USER_PRODUCT_ID/stock/type/seller_warehouse` — write
  seller-warehouse stock (the MLB Multi Origem flow). `meli_facility` (Full) is
  not seller-writable.
- `PUT /user-products/$USER_PRODUCT_ID/stock/type/selling_address` — **MLA / MLC
  only**; not the MLB stock-write mechanism (MLB uses `seller_warehouse`).
- `GET /users/$USER_ID/stores/search?tags=stock_location` — discover the seller's
  stock locations and their `store_id` / `network_node_id`.
- `POST /sites/$SITE/user-products-families/$FAMILY_ID/tasks` — async family
  editor (⚠ verify path/behaviour).

Confirm exact paths and payloads against the live Developers docs — API surface
changes. Items marked "Verified 2026-08-27" were corroborated against
search-indexed copies of the official Developers pages this date (the live pages
return HTTP 403 to bots); everything else here remains `⚠ verify`.

## How to cite

Every reference file ends with a `## Sources` block listing: source title, URL,
origin (Developers / Central de Vendedores / Central de Ajuda / external), last
update date if known, consultation date, and which rules were derived from it.
Summarize — never paste large excerpts of ML documentation.
