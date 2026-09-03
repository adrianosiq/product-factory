# Primary Evidence Registry — Phase 02.3

last_updated: 2026-09-03 (3rd pass — maintainer cleared the 5 HELD artifacts)
status: **35 artifacts registered (SPD-001 … SPD-035); all 35
`COMMITTED_OR_READY`; 0 `HELD`; 0 `SANITIZED`.** All are official Shopee Open
Platform browser print-to-PDF exports (provenance demonstrable from each PDF's
`/Title` = "Documentation / Developer Guide - Shopee Open Platform" and embedded
`open.shopee.com/*` URLs). Raw PDFs live under
`docs/marketplaces/shopee/open-platform/` (see that folder's `INDEX.md`).
**Secret Gate: `PASS`** — see the review history below.

**Primary evidence intake is COMPLETE. This is not P0 documentation-coverage
completion** — `LOGISTICS PRIMARY DOCUMENTATION MISSING` (no `logistics`-service
page was supplied). Artifact safety and P0 coverage are separate dimensions.

**This registration is intake metadata only** — no claim in `claim-registry.md`
has been promoted (that is the resumed Phase 02.3 extraction/reconciliation
pass). "Supports" lists the claims each artifact is *expected to bear on*, not
claims it has been read to confirm.

### Secret-review history

**Pass 1 (2026-09-03, automated):** stdlib text-stream + `/URI` scan of all 35.
30 clean → committed. 5 flagged for review: `authentication.pdf` (an OAuth
`code`+`shop_id` in an example redirect URL) + the 4 §8 credential-context
onboarding pages (image-only, screenshot content unreadable without a renderer).

**Pass 2 (2026-09-03, automated deep non-visual):** zlib-inflated every content
stream; enumerated all `/URI` link annotations, AcroForm fields, annotation
`/Contents`, `Tj`/`TJ` text, image inventory. Result: **all machine-readable
channels of the 5 are clean** (generic Shopee nav URIs only; 0 credential
params; 0 form-field values; 0 real rendered text). The `authentication.pdf`
`code`+`shop_id` is one of **three** example redirect URLs on Shopee's own auth
guide — the other two use explicit `code=xxxxxx` / `shop_id=xxxxxx` /
`main_account_id=xxxxxx` placeholders; the page uses Shopee's canonical doc
`partner_id=10090` (sandbox `1000016`); no `partner_key` / `partner_secret` /
`access_token` value / `refresh_token` value anywhere. But no PDF
rasterizer / OCR was available (`poppler`, `ghostscript`, `imagemagick`,
`pdf.js`, `pip` all absent; installing out of scope), so text baked into the
screenshot images could not be inspected → all 5 stayed `HELD`, Gate `PARTIAL`.

**Pass 3 (2026-09-03) — `MANUAL_VISUAL_REVIEW_BY_MAINTAINER = PASS`.** The
maintainer manually opened and visually inspected the 5 PDFs and confirmed:
credential/key/token/code values visible in them are **`DOCUMENTATION_EXAMPLE`**
values provided by Shopee; **there is currently no Product Factory App created in
the Shopee Open Platform account**, so none of these PDFs contains credentials
generated for or belonging to our application/account. Integrity re-checked:
SHA-256 of all 5 byte-identical to the pre-reorganization originals. The 5
**original, unmodified** PDFs were then added to Git and the temporary
`.gitignore` (which existed solely to hold them) was removed.

| SPD | File | `security_review` | `security_review_method` | credential-shaped values | `artifact_state` |
|---|---|---|---|---|---|
| SPD-001 | `authorization/authentication.pdf` | PASS | MANUAL_VISUAL_REVIEW_BY_MAINTAINER (+ automated non-visual) | `DOCUMENTATION_EXAMPLE` (OAuth `code` / `shop_id` in example redirect URLs; doc `partner_id=10090`) | `COMMITTED_OR_READY` |
| SPD-005 | `getting-started/create-your-app.pdf` | PASS | MANUAL_VISUAL_REVIEW_BY_MAINTAINER | `DOCUMENTATION_EXAMPLE` (no App exists in the maintainer's account → no real Partner Key present) | `COMMITTED_OR_READY` |
| SPD-006 | `getting-started/authorize-your-first-shop.pdf` | PASS | MANUAL_VISUAL_REVIEW_BY_MAINTAINER | `DOCUMENTATION_EXAMPLE` | `COMMITTED_OR_READY` |
| SPD-007 | `getting-started/make-your-first-api-call.pdf` | PASS | MANUAL_VISUAL_REVIEW_BY_MAINTAINER | `DOCUMENTATION_EXAMPLE` | `COMMITTED_OR_READY` |
| SPD-008 | `getting-started/sandbox-testing.pdf` | PASS | MANUAL_VISUAL_REVIEW_BY_MAINTAINER | `DOCUMENTATION_EXAMPLE` | `COMMITTED_OR_READY` |

No PDF was edited / sanitized / redacted / re-exported / recompressed —
the 5 are the immutable originals. No value is stored or reproduced in this
metadata.

**Secret Gate: `PASS`** — all 35 artifacts reviewed; the previously-HELD 5 are
cleared by maintainer visual review. The 30 from pass 1 passed the automated
stdlib text-stream + `/URI` scan (no `partner_key` / token / cookie /
private-key / bearer patterns; only PDF binary/font artefacts and
documentation-example email addresses).

## Artifact state (brief §6)

Every SPD carries one of: `COMMITTED_OR_READY` (in the repo now) · `SANITIZED`
(redacted copy in repo, original held privately) · `HELD` (excluded).

| State | SPDs | Count |
|---|---|---|
| `COMMITTED_OR_READY` | SPD-001 … SPD-035 (all) | 35 |
| `SANITIZED` | *(none)* | 0 |
| `HELD` | *(none)* | 0 |

## Registry

Columns: see "Column meanings" below. `Path` is relative to repo root.
`Cap#` = original download-order prefix (identity is the SPD id, **not** Cap#).
`last_updated` (the page's own revision date) = `UNKNOWN` for all — the PDFs are
image-rendered and no text extractor / OCR was available to read it.

| SPD | Path | Title (verbatim) | Surface | Cap# | Market | API ver | Captured | Auth ctx | Type | Complete | Primary | Supports (claim ids) | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| SPD-001 | `docs/marketplaces/shopee/open-platform/authorization/authentication.pdf` | Developer Guide — Authentication | Developer Guide | 02 | GLOBAL (multi-region: `.com` / `.com.br` / `.cn` auth URLs) | v2 | 2026-09-01 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | SCL-001, SCL-002, SCL-003, SCL-004, SCL-005 | `security_review: PASS` (`MANUAL_VISUAL_REVIEW_BY_MAINTAINER`). OAuth `code`/`shop_id` in example redirect URLs = `DOCUMENTATION_EXAMPLE`. Original, unmodified. |
| SPD-002 | `docs/marketplaces/shopee/open-platform/getting-started/develop-an-integration.pdf` | Developer Guide — Desenvolva uma integração | Developer Guide | — | GLOBAL (captured pt-BR) | n/a | 2026-09-03 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | SCL-010, SCL-142 | Onboarding overview. Has extractable text (~11k chars); text scan clean. |
| SPD-003 | `docs/marketplaces/shopee/open-platform/getting-started/create-a-login.pdf` | Developer Guide — Crie um login | Developer Guide | — | GLOBAL (captured pt-BR) | n/a | 2026-09-03 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | SCL-010 | Image-only; text scan clean. BR-specificity not verified. |
| SPD-004 | `docs/marketplaces/shopee/open-platform/getting-started/create-developer-account.pdf` | Developer Guide — Crie a conta de desenvolvedor | Developer Guide | — | GLOBAL (captured pt-BR) | n/a | 2026-09-03 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | SCL-010, SCL-011 | Image-only; text scan clean. |
| SPD-005 | `docs/marketplaces/shopee/open-platform/getting-started/create-your-app.pdf` | Developer Guide — Crie seu App | Developer Guide | — | GLOBAL (captured pt-BR) | n/a | 2026-09-03 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | SCL-010, SCL-014 | `security_review: PASS` (`MANUAL_VISUAL_REVIEW_BY_MAINTAINER`). No App exists in the account → no real Partner Key present; values are `DOCUMENTATION_EXAMPLE`. |
| SPD-006 | `docs/marketplaces/shopee/open-platform/getting-started/authorize-your-first-shop.pdf` | Developer Guide — Autorize sua primeira loja | Developer Guide | — | GLOBAL (captured pt-BR) | v2 | 2026-09-03 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | SCL-001, SCL-010 | `security_review: PASS` (`MANUAL_VISUAL_REVIEW_BY_MAINTAINER`). Values are `DOCUMENTATION_EXAMPLE`. |
| SPD-007 | `docs/marketplaces/shopee/open-platform/getting-started/make-your-first-api-call.pdf` | Developer Guide — Faça sua primeira chamada de API | Developer Guide | — | GLOBAL (captured pt-BR) | v2 | 2026-09-03 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | SCL-003, SCL-004, SCL-010 | `security_review: PASS` (`MANUAL_VISUAL_REVIEW_BY_MAINTAINER`). Values are `DOCUMENTATION_EXAMPLE`. |
| SPD-008 | `docs/marketplaces/shopee/open-platform/getting-started/sandbox-testing.pdf` | Developer Guide — Realize testes (Sandbox) | Developer Guide | — | GLOBAL (captured pt-BR) | n/a | 2026-09-03 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | SCL-013 | `security_review: PASS` (`MANUAL_VISUAL_REVIEW_BY_MAINTAINER`). Sandbox values are `DOCUMENTATION_EXAMPLE`. |
| SPD-009 | `docs/marketplaces/shopee/open-platform/getting-started/publish-your-app-go-live.pdf` | Developer Guide — Publique seu App (Go Live) | Developer Guide | — | GLOBAL (captured pt-BR) | n/a | 2026-09-03 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | SCL-012, SCL-014 | Image-only; text scan clean. |
| SPD-010 | `docs/marketplaces/shopee/open-platform/product/item/add-item.pdf` | Documentation — v2.product.add_item | API Reference | 04 | GLOBAL | v2 | 2026-09-01 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | SCL-020, SCL-021, SCL-022, SCL-023 | Image-only; text scan clean. |
| SPD-011 | `docs/marketplaces/shopee/open-platform/product/item/get-item-base-info.pdf` | Documentation — v2.product.get_item_base_info | API Reference | 05 | GLOBAL | v2 | 2026-09-01 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | SCL-024, SCL-025, SCL-026, SCL-027 | Image-only; text scan clean. |
| SPD-012 | `docs/marketplaces/shopee/open-platform/product/item/update-item.pdf` | Documentation — v2.product.update_item | API Reference | 21 | GLOBAL | v2 | 2026-09-03 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | SCL-023, SCL-028 | Image-only; text scan clean. |
| SPD-013 | `docs/marketplaces/shopee/open-platform/product/item/unlist-item.pdf` | Documentation — v2.product.unlist_item | API Reference | 22 | GLOBAL | v2 | 2026-09-03 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | SCL-027 | Image-only; text scan clean. |
| SPD-014 | `docs/marketplaces/shopee/open-platform/product/item/delete-item.pdf` | Documentation — v2.product.delete_item | API Reference | 23 | GLOBAL | v2 | 2026-09-03 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | SCL-027 | Image-only; text scan clean. |
| SPD-015 | `docs/marketplaces/shopee/open-platform/product/variation-model/init-tier-variation.pdf` | Documentation — v2.product.init_tier_variation | API Reference | 06 | GLOBAL | v2 | 2026-09-01 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | SCL-030, SCL-031, SCL-032, SCL-033 | Image-only; text scan clean. |
| SPD-016 | `docs/marketplaces/shopee/open-platform/product/variation-model/update-tier-variation.pdf` | Documentation — v2.product.update_tier_variation | API Reference | 17 | GLOBAL | v2 | 2026-09-03 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | SCL-032, SCL-035 | Image-only; text scan clean. |
| SPD-017 | `docs/marketplaces/shopee/open-platform/product/variation-model/add-model.pdf` | Documentation — v2.product.add_model | API Reference | 07 | GLOBAL | v2 | 2026-09-01 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | SCL-030, SCL-031, SCL-034 | Image-only; text scan clean. |
| SPD-018 | `docs/marketplaces/shopee/open-platform/product/variation-model/update-model.pdf` | Documentation — v2.product.update_model | API Reference | 18 | GLOBAL | v2 | 2026-09-03 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | SCL-031 | Image-only; text scan clean. |
| SPD-019 | `docs/marketplaces/shopee/open-platform/product/variation-model/delete-model.pdf` | Documentation — v2.product.delete_model | API Reference | 19 | GLOBAL | v2 | 2026-09-03 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | SCL-031 | Image-only; text scan clean. |
| SPD-020 | `docs/marketplaces/shopee/open-platform/product/variation-model/get-model-list.pdf` | Documentation — v2.product.get_model_list | API Reference | 08 | GLOBAL | v2 | 2026-09-01 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | SCL-031, SCL-033, SCL-141 | Image-only; text scan clean. |
| SPD-021 | `docs/marketplaces/shopee/open-platform/product/category/get-category.pdf` | Documentation — v2.product.get_category | API Reference | 09 | GLOBAL | v2 | 2026-09-01 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | SCL-040, SCL-041, SCL-042 | Image-only; text scan clean. |
| SPD-022 | `docs/marketplaces/shopee/open-platform/product/category/category-recommend.pdf` | Documentation — v2.product.category_recommend | API Reference | 20 | GLOBAL | v2 | 2026-09-03 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | SCL-040 | Image-only; text scan clean. |
| SPD-023 | `docs/marketplaces/shopee/open-platform/product/attribute/get-attribute-tree.pdf` | Documentation — v2.product.get_attribute_tree | API Reference | 10 | GLOBAL | v2 | 2026-09-01 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | SCL-050, SCL-052, SCL-053, SCL-054 | Image-only; text scan clean. |
| SPD-024 | `docs/marketplaces/shopee/open-platform/product/attribute/get-recommend-attribute.pdf` | Documentation — v2.product.get_recommend_attribute | API Reference | 11 | GLOBAL | v2 | 2026-09-01 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | SCL-051 | Image-only; text scan clean. |
| SPD-025 | `docs/marketplaces/shopee/open-platform/product/brand/get-brand-list.pdf` | Documentation — v2.product.get_brand_list | API Reference | 12 | GLOBAL | v2 | 2026-09-01 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | SCL-060, SCL-061, SCL-062 | Image-only; text scan clean. |
| SPD-026 | `docs/marketplaces/shopee/open-platform/product/brand/register-brand.pdf` | Documentation — v2.product.register_brand | API Reference | 13 | GLOBAL | v2 | 2026-09-01 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | SCL-063, SCL-064 | Image-only; text scan clean. |
| SPD-027 | `docs/marketplaces/shopee/open-platform/product/pricing/update-price.pdf` | Documentation — v2.product.update_price | API Reference | 15 | GLOBAL | v2 | 2026-09-01 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | SCL-080, SCL-081, SCL-082 | Image-only; text scan clean. |
| SPD-028 | `docs/marketplaces/shopee/open-platform/product/inventory/update-stock.pdf` | Documentation — v2.product.update_stock | API Reference | 16 | GLOBAL | v2 | 2026-09-01 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | SCL-090, SCL-091, SCL-092, SCL-093, SCL-094 | Image-only; text scan clean. |
| SPD-029 | `docs/marketplaces/shopee/open-platform/product/limits/get-item-limit.pdf` | Documentation — v2.product.get_item_limit | API Reference | 14 | GLOBAL | v2 | 2026-09-01 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | SCL-120, SCL-121, SCL-122, SCL-123 | Image-only; text scan clean. Resolves the Phase 02.2 `get_item_limit` scope question — extraction pending. |
| SPD-030 | `docs/marketplaces/shopee/open-platform/push/push-notifications-webhooks.pdf` | Developer Guide — Push Notifications (Webhooks) | Developer Guide | — | GLOBAL (captured pt-BR) | v2 | 2026-09-03 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | — (webhooks; not a listing-contract claim yet) | Image-only; text scan clean (doc-example emails only). |
| SPD-031 | `docs/marketplaces/shopee/open-platform/policies/sensitive-data.pdf` | Developer Guide — Dados Sensíveis (Sensitive Data) | Developer Guide | — | GLOBAL (captured pt-BR) | n/a | 2026-09-03 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | — (data-handling policy) | Image-only; text scan clean. |
| SPD-032 | `docs/marketplaces/shopee/open-platform/support/faq.pdf` | Developer Guide — Perguntas Frequentes | Developer Guide | — | GLOBAL (captured pt-BR) | n/a | 2026-09-03 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | — | Image-only; text scan clean. Links to `open.shopee.com/announcements/*`. |
| SPD-033 | `docs/marketplaces/shopee/open-platform/support/best-practices-before-opening-a-ticket.pdf` | Developer Guide — Boas práticas antes de abrir um ticket | Developer Guide | — | GLOBAL (captured pt-BR) | n/a | 2026-09-03 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | — | Image-only; text scan clean. |
| SPD-034 | `docs/marketplaces/shopee/open-platform/support/references.pdf` | Developer Guide — Referências | Developer Guide | — | GLOBAL (captured pt-BR) | n/a | 2026-09-03 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | — | Image-only; text scan clean. Links to `open.shopee.com/announcements/*`. |
| SPD-035 | `docs/marketplaces/shopee/open-platform/brazil/br-spi-app-creation-user-guide.pdf` | Developer Guide — BR SPI App Creation User Guide | Developer Guide | — | **BRAZIL** (BR SPI) | n/a | 2026-09-03 | UNKNOWN | print-to-PDF | FULL | VERIFIED_PRIMARY | SCL-010, SCL-011, SCL-012, SCL-014 | Image-only; text scan clean. The only demonstrably BR-specific artifact. |

## Column meanings

| Field | Meaning |
|---|---|
| **SPD** | Stable evidence identity (`SPD-xxx`). Assigned by logical grouping, **not** filesystem order; never renumbered when a file moves. |
| **Path** | Current repo-relative path of the artifact. |
| **Title** | The document / page title. API-Reference rows show the v2 resource name (`/Title` in the PDF is the generic "Documentation - Shopee Open Platform"). Developer-Guide rows show the original page title verbatim. |
| **Surface** | `API Reference` (`open.shopee.com/documents/…`) or `Developer Guide` (`open.shopee.com/developer-guide/…`), from the PDF `/Title`. |
| **Cap#** | Original download-order prefix (`02`…`23`) where the source filename had one; `—` otherwise. **Not** an identity — the SPD id is. |
| **Original locator** | The exact source URL. **`UNKNOWN` for all** — the PDFs are image-rendered and the address bar / canonical link was not captured as extractable text. API-Reference locator *pattern* (Phase 02.2, unverified against these artifacts): `open.shopee.com/documents/v2/v2.product.<method>`. |
| **Market** | `GLOBAL` / `BRAZIL` / `OTHER` / `UNKNOWN_MARKET_SCOPE`. `GLOBAL (captured pt-BR)` = the Developer-Guide journey, served in Portuguese; BR-only scope not verified (image-only, no OCR). |
| **API ver** | `v2` / `v1` / `n/a`. |
| **Captured** | Export date, from the PDF `CreationDate`. |
| **Auth ctx** | Was login required to view the source page. `UNKNOWN` for all (not inferred). Open Platform documentation is generally publicly readable; console/onboarding pages may need login. |
| **Type** | `print-to-PDF` (browser "Save as PDF"; Producer = Skia/PDF) for all 35. |
| **Complete** | `FULL` (whole page printed) for all 35. Most pages render as images rather than selectable text. |
| **Primary** | `VERIFIED_PRIMARY` (PDF `/Title` + embedded `open.shopee.com` URLs establish official Shopee Open Platform provenance) / `CANDIDATE_PRIMARY` / `REJECTED`. |
| **Supports** | Claim ids (`SCL-xxx` from `claim-registry.md`) this artifact is **expected to bear on**. Not a statement that any claim is confirmed. |
| **Notes** | Limitations, HELD status, secret-review notes, scan result. |

## Rules

- **File names prove nothing.** `shopee-official-api.pdf` is not primary until its
  contents establish official Shopee provenance and the maintainer records where
  it came from.
- A copied API page **can** be primary if the content clearly identifies the
  official Shopee developer source and the origin is recorded.
- **Never** commit `partner_key`, `access_token`, `refresh_token`, authorization
  `code`, cookies, session ids, client secrets, private keys, or seller PII.
  Redact before storage; if artifacts cannot be safely committed, keep them
  out-of-repo and record only this metadata.
- Screenshots → `completeness = SCREENSHOT`; do not infer fields outside the
  frame, and do not reconstruct missing portions from community SDKs.
