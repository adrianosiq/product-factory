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
| User Products | https://developers.mercadolivre.com.br/pt_br/user-products | 2026-06-17 (per requester) ⚠ verify | product-structure, variations-and-user-products, titles-and-family-name |
| Preço por variação | https://developers.mercadolivre.com.br/pt_br/preco-variacao | 2026 ⚠ verify | variations-and-user-products, pricing-and-commercial |
| Variations (modelo tradicional) | https://developers.mercadolivre.com.br/pt_br/variacoes | ⚠ verify | variations-and-user-products |
| Categorias e Atributos | https://developers.mercadolivre.com.br/pt_br/categorias-e-atributos-veiculos | ⚠ verify | categories, attributes |
| Preditor de categorias | https://developers.mercadolivre.com.br/pt_br/categorizacao-de-produtos | ⚠ verify | categories |
| Validações / Validador de publicações | https://developers.mercadolivre.com.br/pt_br/validacoes | ⚠ verify | quality-audit, categories |
| Identificadores de produtos (GTIN/EAN) | https://developers.mercadolivre.com.br/pt_br/identificadores-de-produtos | ⚠ verify | attributes |
| Trabalhar com imagens | https://developers.mercadolivre.com.br/pt_br/trabalhar-com-imagens | 2026-03-24 (per requester) ⚠ verify | images |
| Descrição de produtos | https://developers.mercadolivre.com.br/pt_br/descricao-de-produtos | 2026-03-13 (per requester) ⚠ verify | descriptions |
| Compatibilidades de autopeças | https://developers.mercadolivre.com.br/pt_br/compatibilidades-itens-e-produtos-de-autopecas | ⚠ verify | attributes (compatibility) |
| Informe compatibilidades | https://developers.mercadolivre.com.br/informe-compatibilidades | ⚠ verify | attributes (compatibility) |
| Sincronização e modificação de publicações | https://developers.mercadolivre.com.br/pt_br/produto-sincronizacao-de-publicacoes | ⚠ verify | product-structure |
| Como criar anúncios eficientes | https://vendedores.mercadolivre.com.br/nota/como-criar-anuncios-eficientes-no-mercado-livre | 2026 ⚠ verify | seo-and-discovery, titles-and-family-name, images |
| Como criar um título atrativo | https://vendedores.mercadolivre.com.br/nota/como-criar-um-titulo-atrativo | 2026 ⚠ verify | titles-and-family-name |
| O status das suas fichas técnicas | https://vendedores.mercadolivre.com.br/notas/o-status-das-suas-fichas-tecnicas/ | 2026 ⚠ verify | attributes, seo-and-discovery |
| Códigos universais | https://www.mercadolivre.com.br/codigos-universais | ⚠ verify | attributes |
| Catálogo / Buy Box (Central) | https://vendedores.mercadolivre.com.br/ (buscar "catálogo") | ⚠ verify | catalog |

## API endpoints referenced by this Skill (DYNAMIC)

- `GET /sites/MLB/domain_discovery/search?q=...` — category / domain prediction
- `GET /categories/$CATEGORY_ID` — settings incl. `settings.max_pictures_per_item`,
  `settings.max_pictures_per_item_var`, `settings.max_title_length`,
  `settings.listing_allowed`, variation-related flags
- `GET /categories/$CATEGORY_ID/attributes` — attribute ids, `values`, `tags`
  (`required`, `new_required`, `conditional_required`), `PARENT_PK`, `CHILD_PK`
- `GET /categories/$CATEGORY_ID/attributes/conditional`
- `GET /products/$CATALOG_PRODUCT_ID` and catalog search — catalog match
- `POST /items/validate` (pre-publish validation)
- `POST /items/{MLB}/compatibilities`
- `GET /items/{MLB}/description` / `POST` with `plain_text`

Confirm exact paths and payloads against the live Developers docs — API surface
changes and this list is `⚠ verify`.

## How to cite

Every reference file ends with a `## Sources` block listing: source title, URL,
origin (Developers / Central de Vendedores / Central de Ajuda / external), last
update date if known, consultation date, and which rules were derived from it.
Summarize — never paste large excerpts of ML documentation.
