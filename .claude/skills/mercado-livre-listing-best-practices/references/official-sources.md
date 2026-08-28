# Official sources

last_reviewed: 2026-08-27
fetch_method_note: >-
  Direct fetch of developers.mercadolivre.com.br and vendedores.mercadolivre.com.br
  was blocked (HTTP 403 bot protection) when this Skill was authored. Content below
  is reconstructed from search-engine summaries of those official pages plus dates
  provided by the requester. Every rule derived from these pages is tagged
  "⚠ verify" until a maintainer re-reads the live page.

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
| User Products | https://developers.mercadolivre.com.br/pt_br/user-products | 2026-06-17 (per requester); `family_name` rules verified 2026-08-27 (search-indexed; live 403) | product-structure, variations-and-user-products, titles-and-family-name |
| Preço por variação | https://developers.mercadolivre.com.br/pt_br/preco-variacao | 2026 ⚠ verify | variations-and-user-products, pricing-and-commercial |
| Variations (modelo tradicional) | https://developers.mercadolivre.com.br/pt_br/variacoes | ⚠ verify | variations-and-user-products |
| Categorias e Atributos | https://developers.mercadolivre.com.br/pt_br/categorias-e-atributos-veiculos | ⚠ verify | categories, attributes |
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
- `GET /products/$CATALOG_PRODUCT_ID` and catalog search — catalog match
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

Confirm exact paths and payloads against the live Developers docs — API surface
changes. Items marked "Verified 2026-08-27" were corroborated against
search-indexed copies of the official Developers pages this date (the live pages
return HTTP 403 to bots); everything else here remains `⚠ verify`.

## How to cite

Every reference file ends with a `## Sources` block listing: source title, URL,
origin (Developers / Central de Vendedores / Central de Ajuda / external), last
update date if known, consultation date, and which rules were derived from it.
Summarize — never paste large excerpts of ML documentation.
