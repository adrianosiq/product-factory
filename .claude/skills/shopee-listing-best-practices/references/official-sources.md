# Official sources — registry & verification quality

last_reviewed: 2026-08-28
volatile: true
fetch_method_note: >-
  As of last_reviewed, NO Shopee source in this file has been read LIVE.
  `open.shopee.com` (Open Platform API portal) was unreachable during Phase 01
  discovery. `seller.shopee.com.br/edu`, `help.shopee.com.br` and
  `ads.shopee.com.br/learn` render client-side and return only a page title to a
  fetch — their bodies are known only from search snippets. Every row below is
  therefore `SEARCH_INDEXED` or `UNVERIFIED`. A `SEARCH_INDEXED` row is not
  `LIVE` and may be stale or wrong; confirm before locking any rule.

## How to read this file

- **Source authority** and **verification quality** are two independent axes.
  A row can be `OFFICIAL` authority + `SEARCH_INDEXED` quality. Do not collapse
  them.
- **Verification quality** is one of exactly three values:

  | Value | Meaning |
  |---|---|
  | `LIVE` | The primary page was read directly this cycle. **None yet.** |
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

## Source registry

| # | Source | Scope | Claims it supports | Verification quality | Verified date | Notes |
|---|---|---|---|---|---|---|
| S1 | Shopee Open Platform API docs — `https://open.shopee.com` | `GLOBAL_API` | intended source for every v2 endpoint / schema / limit | `UNVERIFIED` | 2026-08-28 | **UNREACHABLE** from tooling; not usefully indexed. Nothing read. All v2 facts elsewhere are reconstructed. |
| S2 | Centro de Educação do Vendedor — `https://seller.shopee.com.br/edu` | `BRAZIL` | image count/ratio, cover-photo rules, Title Case guidance, brand mandatory + "Sem marca", restricted-item authorisation flow | `SEARCH_INDEXED` | 2026-08-28 | Page = title only on fetch; body via snippets. Arts. cited: 17369 (3:4 images), 3304 (prohibited/restricted), 12544 (restricted-item release), 10619 (brand), 20631 (Shopee Video). |
| S3 | Central de Ajuda — `https://help.shopee.com.br` | `BRAZIL` | prohibited vs restricted split; regulators ANVISA/ANATEL/INMETRO/MAPA/ANS; homologation-absent → prohibited; Shopee Live policy | `SEARCH_INDEXED` | 2026-08-28 | Art. 76226 (Produtos Proibidos e Restritos); art. 188686 (Shopee Live prohibited products). |
| S4 | Shopee Ads education — `https://ads.shopee.com.br/learn` | `BRAZIL` | listing-quality *recommendations*; creative guidelines; video duration | `SEARCH_INDEXED` | 2026-08-28 | Recommendations only — **not** organic requirements, **not** proven ranking factors. |
| S5 | "Pontos de Penalidade" — official Shopee BR PDF (`deo.shopeemobile.com/.../Pontos de Penalidade.pdf`) | `BRAZIL` | penalty-point table: weekly accrual, 60-day validity, per-infraction points, progressive sanctions | `SEARCH_INDEXED` | 2026-08-28 | PDF referenced, not parsed. Numeric values `⚠ verify`. |
| S6 | GitHub `teacat/shopeego` (Go, **v1**) | `GLOBAL_API` | request/response struct **names**; `tier_index` combination semantics; `condition` / `days_to_ship` / pre-order 7–30 | `SEARCH_INDEXED` | 2026-08-28 | **v1 API** — field names only; limits stale, do not trust numbers. |
| S7 | GitHub `wjp-letgo/shopeego/product` (Go, v2) | `GLOBAL_API` | v2 endpoint **names**; `item_status ∈ NORMAL/BANNED/UNLIST/DELETED` | `SEARCH_INDEXED` | 2026-08-28 | Names only; no schema. Not a contract. |
| S8 | GitHub `raviMukti/shopee-api-client`, `mu-hanz/shoapi` (PHP, v2) | `GLOBAL_API` | host `partner.shopeemobile.com` (prod) / `partner.test-stable.shopeemobile.com` (sandbox); token 4 h; refresh 30 d | `SEARCH_INDEXED` | 2026-08-28 | Community SDKs. Auth timing `⚠ verify`. |
| S9 | `developer.inlinex.com.sg`, `rollout.com`, `api2cart.com` — integrator guides | `GLOBAL_API` | v2 paths; sign base `partner_id+path+timestamp+access_token+shop_id` HMAC-SHA256; timestamp in seconds; "store image ids not URLs" | `SEARCH_INDEXED` | 2026-08-28 | Third-party. Signature composition `⚠ verify`. |
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

Per-row fields to maintain (brief §43): `id`, `authority` (A–E tier),
`scope` (`GLOBAL_API` / `BRAZIL` / `OTHER_MARKET_REFERENCE` /
`UNRESOLVED_FOR_BRAZIL`), `verification_quality` (`LIVE` / `SEARCH_INDEXED` /
`UNVERIFIED`), `verified_at`, `supports`, `staleness` ("re-verify before
integration").

| id | authority | scope | verification | verified_at | supports | staleness |
|---|---|---|---|---|---|---|
| `open.shopee.com/documents/v2/*` (doc locators) | A | `GLOBAL_API` | `UNVERIFIED` (tool-blocked + SPA) | — | the pages that must replace every reconstructed row | re-verify at integration — mandatory |
| v2 `product` method set (S12+S13) | E | `GLOBAL_API` / `UNRESOLVED_FOR_BRAZIL` | `SEARCH_INDEXED` · MEDIUM · MULTI_SOURCE (non-primary — not `CONFIRMED`) | 2026-08-28 | endpoint names, paths, key params, lifecycle order — a corroborated contract candidate | re-verify before integration |
| auth model (S12+S14+S15) | D/E | `GLOBAL_API` | `SEARCH_INDEXED` · MEDIUM · MULTI_SOURCE | 2026-08-28 | OAuth flow, token lifetimes, `/api/v2`, HMAC-SHA256 — **exact signing/base string `UNVERIFIED`** | re-verify signing before any client |
| `openplatform.shopee.com.br` host string (S12 region config) | E | `BRAZIL` | `SEARCH_INDEXED` · LOW · SINGLE_SOURCE | 2026-08-28 | an existence signal for a BR Open Platform surface — **not** verified as the canonical Product API base URL | corroborate; confirm which host serves BR Product API |
| BR Open Platform application/integration (S16) | B | `BRAZIL` | `SEARCH_INDEXED` (title/snippet) | 2026-08-28 | BR sellers have a documented Open Platform application path | fetch article bodies |
| `get_item_limit` scope | E | `GLOBAL_API` | `SEARCH_INDEXED`, **conflicting** | 2026-08-28 | possibly a shop listing *quota*, not field limits | resolve before locking numeric limits |

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
