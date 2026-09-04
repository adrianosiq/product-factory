# Official sources — registry & verification quality

last_reviewed: 2026-08-28
phase_02_3_reviewed: 2026-09-03
volatile: true
fetch_method_note: >-
  **Phase 02.3 (2026-09-03): 35 Shopee Open Platform pages WERE read LIVE** —
  see the "PRIMARY source registry" section below (SPD-001…SPD-035, stored at
  `docs/marketplaces/shopee/open-platform/`). The `S1…S17` rows further down are
  the older `SEARCH_INDEXED` / `UNVERIFIED` reconstruction and are **superseded**
  wherever a PRIMARY row covers the same fact. `seller.shopee.com.br/edu`,
  `help.shopee.com.br` and `ads.shopee.com.br/learn` (S2–S4) still render
  client-side — image pixel rules, prohibited-products policy and penalty
  mechanics remain `SEARCH_INDEXED`. The `LIVE` cell in the table below now
  reads "SPD-* — read 2026-09-03" for the covered surfaces.

## How to read this file

- **Source authority** and **verification quality** are two independent axes.
  A row can be `OFFICIAL` authority + `SEARCH_INDEXED` quality. Do not collapse
  them.
- **Verification quality** is one of exactly three values:

  | Value | Meaning |
  |---|---|
  | `LIVE` | The primary page was read directly. `SPD-001`…`SPD-035` (Phase 02.3, 2026-09-03). |
  | `SEARCH_INDEXED` | Reconstructed from search-engine snippets, third-party integrator docs, or community SDK source. Plausible, not confirmed. |
  | `UNVERIFIED` | Not corroborated at all — a schema/limit/behaviour only guessed at, or an open question. |

- **Priority order** when sources conflict: (1) Shopee Open Platform API docs
  (`open.shopee.com`) → (2) Centro de Educação do Vendedor
  (`seller.shopee.com.br/edu`) → (3) Central de Ajuda (`help.shopee.com.br`) →
  (4) other official Shopee pages → (5) everything else (integrators, SDKs,
  educators) — **strategy input only, never a source of an `OFFICIAL` rule.**
- Community SDK endpoint names are **not** a canonical API contract. If a v2
  endpoint name comes only from an SDK or an integrator blog, its schema is
  `UNVERIFIED`.

## PRIMARY source registry (Phase 02.3 — 2026-09-03)

35 official **Shopee Open Platform** pages, captured by the maintainer as
browser print-to-PDF, provenance-verified, secret-reviewed, and **read** on
2026-09-03. Stored at `docs/marketplaces/shopee/open-platform/`; full registry
+ per-fact `SPD` citations in `research/shopee-primary-docs/`
(`evidence-registry.md`, `claim-registry.md`, `extraction-report.md`,
`reconciliation-report.md`).

| id range | Surface | Verification | Covers |
|---|---|---|---|
| `SPD-001` | Developer Guide — Authorization & Authentication | **LIVE / PRIMARY_VERIFIED** | auth flow, hosts (incl. BR `openplatform.shopee.com.br`), sign base string per API type, token lifetimes (4h / 30d / 10min / 5min / ≤360d) |
| `SPD-002`…`SPD-009`, `SPD-035` | Developer Guide — BR developer journey + BR SPI App guide | **LIVE / PRIMARY_VERIFIED** | Brazil eligibility: dev-profile approval + Go-Live review; BR developer types (Registered Business Seller / ISV); app-category permission model; Sandbox; whitelist scope |
| `SPD-010`…`SPD-014` | API Reference — `product` item lifecycle | **LIVE / PRIMARY_VERIFIED** | `add_item` (required/optional fields, error family), `get_item_base_info` (`item_id_list` ≤ 50, `item_status` enum, `stock_info_v2`, `ssp_id`), `update_item` / `unlist_item` / `delete_item` |
| `SPD-015`…`SPD-020` | API Reference — `product` variations / models | **LIVE / PRIMARY_VERIFIED** | max 2 tiers, `standardise_tier_variation`, `model_id = 0` = no-model, `model_status`, model/option counts |
| `SPD-021`…`SPD-024` | API Reference — `get_category` / `category_recommend` / `get_attribute_tree` / `get_recommend_attribute` | **LIVE / PRIMARY_VERIFIED** | tree + `has_children` (leaf), `mandatory` attribute flag, `input_type` enum |
| `SPD-025`, `SPD-026` | API Reference — `get_brand_list` / `register_brand` | **LIVE / PRIMARY_VERIFIED** | per-leaf brands, `status` 1/2, brand object required in `add_item`, `brand_id: 0` = No Brand, QC-reviewed registration |
| `SPD-027`, `SPD-028`, `SPD-029` | API Reference — `update_price` / `update_stock` / `get_item_limit` | **LIVE / PRIMARY_VERIFIED** | batch 1–50, `get_item_limit` = the dynamic source of title/desc/image/price/stock/tier/DTS limits + weight/dimension/size-chart/GTIN requiredness; `gtin_validation_rule` Mandatory/Flexible/Optional; `seller_stock` absolute write |
| `SPD-030`…`SPD-034` | Developer Guide — webhooks / sensitive-data / FAQ / references / ticket best-practices | `PRIMARY` (no listing-contract claims) | — |

`PRIMARY_NOT_FOUND` in the corpus (need acquisition): `v2.logistics.get_channel_list`,
`v2.logistics.get_address(_list)`, `v2.shop.get_warehouse_detail`,
`v2.media_space.upload_image` / `upload_video`, `v2.product.search_attribute_value_list`,
a dedicated `get_dts_limit` page, `update_item` full field list, penalty /
content-diagnosis / violation APIs.

## Source registry

| # | Source | Scope | Claims it supports | Verification quality | Verified date | Notes |
|---|---|---|---|---|---|---|
| S1 | Shopee Open Platform API docs — `https://open.shopee.com` | `GLOBAL_API` | intended source for every v2 endpoint / schema / limit | `UNVERIFIED` | 2026-08-28 | **UNREACHABLE** from tooling; not usefully indexed. Nothing read. All v2 facts elsewhere are reconstructed. |
| S2 | Centro de Educação do Vendedor — `https://seller.shopee.com.br/edu` | `BRAZIL` | image count/ratio, cover-photo rules, Title Case guidance, brand mandatory + "Sem marca", restricted-item authorisation flow | `SEARCH_INDEXED` | 2026-08-28 | Page = title only on fetch; body via snippets. Arts. cited: 17369 (3:4 images), 3304 (prohibited/restricted), 12544 (restricted-item release), 10619 (brand), 20631 (Shopee Video). |
| S3 | Central de Ajuda — `https://help.shopee.com.br` | `BRAZIL` | prohibited vs restricted split; regulators ANVISA/ANATEL/INMETRO/MAPA/ANS; homologation-absent → prohibited; Shopee Live policy | `SEARCH_INDEXED` | 2026-08-28 | Art. 76226 (Produtos Proibidos e Restritos); art. 188686 (Shopee Live prohibited products). |
| S4 | Shopee Ads education — `https://ads.shopee.com.br/learn` | `BRAZIL` | listing-quality *recommendations*; creative guidelines; video duration | `SEARCH_INDEXED` | 2026-08-28 | Recommendations only — **not** organic requirements, **not** proven ranking factors. |
| S5 | "Pontos de Penalidade" — official Shopee BR PDF (`deo.shopeemobile.com/.../Pontos de Penalidade.pdf`) | `BRAZIL` | penalty-point table: weekly accrual, 60-day validity, per-infraction points, progressive sanctions | `SEARCH_INDEXED` | 2026-08-28 | PDF referenced, not parsed. Numeric values `⚠ verify`. |
| S6 | GitHub `teacat/shopeego` (Go, **v1**) | `GLOBAL_API` | request/response struct **names**; `tier_index` combination semantics; `condition` / `days_to_ship` / pre-order 7–30 | `SEARCH_INDEXED` | 2026-08-28 | **v1 API** — field names only; limits stale, do not trust numbers. |
| S7 | GitHub `wjp-letgo/shopeego/product` (Go, v2) | `GLOBAL_API` | v2 endpoint **names**; `item_status` guess incl. plain `DELETED` — **corrected by SPD-011** to `NORMAL/BANNED/UNLIST/SELLER_DELETE/SHOPEE_DELETE/REVIEWING` | `SEARCH_INDEXED` | 2026-08-28 | Names only; no schema. Not a contract. |
| S8 | GitHub `raviMukti/shopee-api-client`, `mu-hanz/shoapi` (PHP, v2) | `GLOBAL_API` | host `partner.shopeemobile.com` (prod) / `partner.test-stable.shopeemobile.com` (sandbox); token 4 h; refresh 30 d | `SEARCH_INDEXED` | 2026-08-28 | Community SDKs. Auth timing `⚠ verify`. |
| S9 | `developer.inlinex.com.sg`, `rollout.com`, `api2cart.com` — integrator guides | `GLOBAL_API` | v2 paths; sign base `partner_id+path+timestamp+access_token+shop_id` HMAC-SHA256; timestamp in seconds; "store image ids not URLs" | `SEARCH_INDEXED` | 2026-08-28 | Third-party. Signature composition now **`PRIMARY_VERIFIED`** per API type (SPD-001, `api-and-auth.md` §0). |
| S10 | `base.com`, `anymarket`, `ideris`, `maino`, `gobots`, `destraveescale`, `1001clicks`, `mambadigital` — BR integrators / educators | `BRAZIL` | title ≈255–256; description ≈5,000; image min ≈350×350, rec ≈1024×1024; EAN "obrigatório para alguns produtos"; brand mandatory attribute; 2026 stricter image moderation; penalty thresholds | `SEARCH_INDEXED` | 2026-08-28 | Numbers are LOW–MEDIUM confidence. **Never hardcode.** |
| S11 | `blog.gs1br.org` — GS1 Brasil | `BRAZIL` | EAN/GTIN as best practice, **not** a blanket Shopee listing requirement | `SEARCH_INDEXED` | 2026-08-28 | Counterpoint to S10 on EAN requiredness. |
| S12 | GitHub `QuoVadis86/shopee-sdk` (Go, "380+ endpoints") | `GLOBAL_API` | v2 `product` method names; region hosts incl. **`openplatform.shopee.com.br`**; dedicated `br` service (`query_shop_enrollment_status`, `query_shop_invoice_error`, block-status); sign base string | `SEARCH_INDEXED` | 2026-08-28 (Phase 02.2) | Community SDK. Names only. **Not** a contract. |
| S13 | GitHub `congminh1254/shopee-sdk` (TS, "100% coverage") — `docs/managers/product.md`, `schemas/v2.product.*.json` | `GLOBAL_API` | ~25 v2 `product` methods → path + required params + an `open.shopee.com/documents/v2/...?module=89` locator; `product` vs `global_product` split | `SEARCH_INDEXED` | 2026-08-28 (Phase 02.2) | Highest-quality triangulation. Locators **not read**. |
| S14 | `rollout.com` — Shopee integration guide | `GLOBAL_API` | `/api/v2` base; `get_item_base_info` param `product_id`, response `item_id`; `auth/token/get`; token 4 h | `SEARCH_INDEXED` | 2026-08-28 (Phase 02.2) | Third-party. |
| S15 | `publicapis.io/shopee-api`, `api2cart.com` | `GLOBAL_API` | `/api/v2` base; sandbox host; `partner_id`+`partner_key`+`shop_id`+`access_token`; HMAC-SHA256; "route to the correct country endpoint" | `SEARCH_INDEXED` | 2026-08-28 (Phase 02.2) | Third-party. |
| S16 | Centro de Educação do Vendedor — arts. **3445** ("Shopee Open API Platform \| Passo a Passo de Solicitação"), **27314** ("Open Platform Shopee: Guia Prático de Integração") | `BRAZIL` | Shopee BR **documents an Open Platform application + integration process** for sellers / integrators | `SEARCH_INDEXED` | 2026-08-28 (Phase 02.2) | Title + snippet only; body is SPA. Best BR-availability evidence to date. |
| S17 | `geckoapi.com.br` (BR scraping-API blog, 2026) | `BRAZIL` | BR store operations use "Open Platform … Aplicação e autorização conforme o programa oficial" | `SEARCH_INDEXED` | 2026-08-28 (Phase 02.2) | Not the Open Platform itself. Framing only. |

## API resource registry

The full endpoint list, auth model, BR-access status and the dynamic-check
registry are in `references/api-and-auth.md`; the Phase 02.2 analysis is in
`research/shopee-api-contract/phase-02.2-report.md` (§7, §15, §27, §29).

**The rows below are the Phase 02.2 reconstruction.** They are **superseded by
the "PRIMARY source registry" section above** (and `api-and-auth.md` §0)
wherever a PRIMARY row covers the same fact.

Per-row fields to maintain (brief §43): `id`, `authority` (A–E tier),
`scope` (`GLOBAL_API` / `BRAZIL` / `OTHER_MARKET_REFERENCE` /
`UNRESOLVED_FOR_BRAZIL`), `verification_quality` (`LIVE` / `SEARCH_INDEXED` /
`UNVERIFIED`), `verified_at`, `supports`, `staleness` ("re-verify before
integration").

| id | authority | scope | verification | verified_at | supports | staleness |
|---|---|---|---|---|---|---|
| `open.shopee.com/documents/v2/*` (doc locators) | A | `GLOBAL_API` | `UNVERIFIED` (tool-blocked + SPA) | — | the pages that must replace every reconstructed row | re-verify at integration — mandatory |
| v2 `product` method set | A | `GLOBAL_API` (BR hosts on every page) | **`PRIMARY_VERIFIED`** (SPD-010…SPD-029) — superseded the S12+S13 reconstruction | 2026-09-03 | endpoint names, paths, key params, required fields, error families, lifecycle order | re-verify quarterly (volatile) |
| auth model | A | `GLOBAL_API` | **`PRIMARY_VERIFIED`** (SPD-001) — OAuth flow, token lifetimes (4h/30d/10min/5min/≤360d), `/api/v2`, HMAC-SHA256, **sign base string per API type** (§0) | 2026-09-03 | full auth + signing contract | re-verify quarterly |
| `openplatform.shopee.com.br` host | A | `BRAZIL` | **`PRIMARY_VERIFIED`** (SPD-001, SPD-010…029) — the Brazil production base URL `https://openplatform.shopee.com.br/api/v2/<path>` on every API-ref page | 2026-09-03 | BR production Product API base URL | re-verify quarterly |
| BR Open Platform onboarding | A | `BRAZIL` | **`PRIMARY_VERIFIED`** flow (SPD-002…SPD-009, SPD-035); developer-profile approval **criteria** `PRIMARY_NOT_FOUND` | 2026-09-03 | documented BR developer journey; eligibility model | acquire the approval-criteria page |
| `get_item_limit` scope | A | `GLOBAL_API` | **`PRIMARY_VERIFIED`** (SPD-029) — the dynamic source of title/desc/image/price/stock/tier/DTS limits + weight/dimension/size-chart/GTIN requiredness; `item_count_limit` is the separate shop quota field | 2026-09-03 | the dynamic-limit resolution source | re-verify quarterly |

## Re-verification triggers

Re-fetch the primary pages (S1–S3, S5) and update the rows when:

- any Shopee-specific rule in a reference file is still tagged `⚠ verify` and it
  is about to drive a hard decision, OR
- a reference file's `last_reviewed` is older than one quarter, OR
- Phase 02.3 (Primary Documentation Ingestion) obtains portal / sandbox access
  or user-supplied doc exports (then transcribe S1 / S16 and flip rows to
  `LIVE`).

Phase 02.2 (2026-08-28) added S12–S17 and corrected several Phase 02.1 API
details but read **nothing** `LIVE` — `open.shopee.com`, `developer.shopee.com`
and `web.archive.org` are blocked at the fetch-tool level and the portal is a
client-rendered SPA.

## How to cite

Every reference file ends with a `## Sources` block: source title, URL, origin
(Open Platform / Centro de Educação / Central de Ajuda / external), update date
if known, consultation date, verification quality, and which rules were derived
from it. Summarize — never paste large excerpts of Shopee documentation.

## Sources

- Shopee Open Platform API portal — `https://open.shopee.com` — Open Platform —
  consulted 2026-08-28 — `UNVERIFIED` (UNREACHABLE) — intended source for every
  v2 endpoint/schema; nothing read.
- Centro de Educação do Vendedor Shopee BR — `https://seller.shopee.com.br/edu`
  — Centro de Educação — consulted 2026-08-28 — `SEARCH_INDEXED` — image / title
  / brand / restricted-item rules (body via snippets only).
- Central de Ajuda Shopee BR — `https://help.shopee.com.br` (art. 76226; art.
  188686) — Central de Ajuda — consulted 2026-08-28 — `SEARCH_INDEXED` —
  prohibited vs restricted tiers; regulator list.
- Shopee Ads BR education — `https://ads.shopee.com.br/learn` — Shopee Ads —
  consulted 2026-08-28 — `SEARCH_INDEXED` — listing-quality recommendations only.
- "Pontos de Penalidade" (Shopee BR PDF) — `deo.shopeemobile.com/.../Pontos de
  Penalidade.pdf` — Shopee BR — consulted 2026-08-28 — `SEARCH_INDEXED`
  (referenced, not parsed) — penalty-point mechanics.
- Community SDKs & integrator guides (`teacat/shopeego`, `wjp-letgo/shopeego`,
  `raviMukti/shopee-api-client`, `mu-hanz/shoapi`, `developer.inlinex.com.sg`,
  `rollout.com`, `api2cart.com`) — external — consulted 2026-08-28 —
  `SEARCH_INDEXED` — v2 endpoint names, host URLs, auth outline. **Not** a
  canonical contract.
- BR integrators / educators (`base.com`, `anymarket`, `ideris`, `maino`,
  `gobots`, `destraveescale`, `1001clicks`, `mambadigital`, `blog.gs1br.org`) —
  external — consulted 2026-08-28 — `SEARCH_INDEXED` — provisional numeric
  limits and EAN framing; strategy input only.
- Full detail: `research/shopee-listing-skill/discovery-report.md` §2, §Sources.
