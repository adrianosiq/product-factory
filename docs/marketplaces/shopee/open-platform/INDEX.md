# Shopee Open Platform — primary documentation (navigational index)

**What this is:** an immutable set of official **Shopee Open Platform** pages,
captured 2026-09-01 / 2026-09-03 by browser print-to-PDF from `open.shopee.com`.
Used as **primary technical evidence** for the Product Factory Shopee Skills.

**This file answers "where is the doc I need?"** — it is a catalog, not the
provenance record.

| Concern | File |
|---|---|
| Navigational catalog (this file) | `docs/marketplaces/shopee/open-platform/INDEX.md` |
| Provenance + scope + primary status + what each artifact supports | `research/shopee-primary-docs/evidence-registry.md` |
| Technical claims + which evidence supports them | `research/shopee-primary-docs/claim-registry.md` |

Do not edit the PDFs. Do not rename without updating both this file and the
evidence registry (SPD id is the stable identity; the path may change).

## Status

- **30 of 35 PDFs are committed.** **5 remain HELD** and are git-ignored (see
  `.gitignore` at repo root and the table below). 2nd-pass review (2026-09-03):
  a deep non-visual scan (all link URIs, form fields, rendered text, image
  inventory) of the 5 is **clean** — and the one credential-shaped value
  detected earlier (an OAuth `code` in `authentication.pdf`) is a
  **documentation example** (it sits beside `code=xxxxxx` placeholder variants on
  Shopee's own auth guide). But all 5 are screenshot-heavy credential-context
  pages, and **no PDF renderer / OCR is available here** to inspect the image
  content, so none can be cleared. A maintainer with a PDF viewer should eyeball
  them. **Secret Gate: PARTIAL.**
- No claim in `claim-registry.md` has been promoted to `PRIMARY_VERIFIED` yet —
  extraction / reconciliation is the resumed Phase 02.3 pass, not this
  organization pass.

## Quick lookup

| Need | Go to |
|---|---|
| `add_item`, `update_item`, `get_item_base_info`, `unlist_item`, `delete_item` | `product/item/` |
| tier variations & models (`init_tier_variation`, `add_model`, `update_model`, `delete_model`, `get_model_list`, `update_tier_variation`) | `product/variation-model/` |
| `get_category`, `category_recommend` | `product/category/` |
| `get_attribute_tree`, `get_recommend_attribute` | `product/attribute/` |
| `get_brand_list`, `register_brand` | `product/brand/` |
| `update_price` | `product/pricing/` |
| `update_stock` | `product/inventory/` |
| `get_item_limit` (shop listing quota — scope TBD) | `product/limits/` |
| authentication / signing / tokens | `authorization/` *(HELD — see below)* |
| developer journey (login → dev account → app → authorize shop → first call → sandbox → Go Live) | `getting-started/` |
| Brazil app creation (BR SPI) | `brazil/` |
| webhooks / push notifications | `push/` |
| sensitive-data policy | `policies/` |
| FAQ, references, ticket best-practices | `support/` |

## Catalog

| SPD | Document | Path (under `docs/marketplaces/shopee/open-platform/`) | Surface | Market | Captured | Cap# | API resource / topic |
|---|---|---|---|---|---|---|---|
| SPD-001 | Authentication *(HELD)* | `authorization/authentication.pdf` | Developer Guide | GLOBAL | 2026-09-01 | 02 | auth flow, request signing, `access_token` / `refresh_token` |
| SPD-002 | Develop an integration | `getting-started/develop-an-integration.pdf` | Developer Guide | GLOBAL (pt-BR) | 2026-09-03 | — | integration overview / developer lifecycle |
| SPD-003 | Create a login | `getting-started/create-a-login.pdf` | Developer Guide | GLOBAL (pt-BR) | 2026-09-03 | — | Shopee account login |
| SPD-004 | Create developer account | `getting-started/create-developer-account.pdf` | Developer Guide | GLOBAL (pt-BR) | 2026-09-03 | — | developer-account registration |
| SPD-005 | Create your App *(HELD)* | `getting-started/create-your-app.pdf` | Developer Guide | GLOBAL (pt-BR) | 2026-09-03 | — | App creation, `partner_id` / `partner_key` |
| SPD-006 | Authorize your first shop *(HELD)* | `getting-started/authorize-your-first-shop.pdf` | Developer Guide | GLOBAL (pt-BR) | 2026-09-03 | — | shop OAuth authorization |
| SPD-007 | Make your first API call *(HELD)* | `getting-started/make-your-first-api-call.pdf` | Developer Guide | GLOBAL (pt-BR) | 2026-09-03 | — | first signed request walkthrough |
| SPD-008 | Sandbox testing *(HELD)* | `getting-started/sandbox-testing.pdf` | Developer Guide | GLOBAL (pt-BR) | 2026-09-03 | — | sandbox / test environment |
| SPD-009 | Publish your App (Go Live) | `getting-started/publish-your-app-go-live.pdf` | Developer Guide | GLOBAL (pt-BR) | 2026-09-03 | — | app review / production go-live |
| SPD-010 | add_item | `product/item/add-item.pdf` | API Reference | GLOBAL | 2026-09-01 | 04 | `v2.product.add_item` |
| SPD-011 | get_item_base_info | `product/item/get-item-base-info.pdf` | API Reference | GLOBAL | 2026-09-01 | 05 | `v2.product.get_item_base_info` |
| SPD-012 | update_item | `product/item/update-item.pdf` | API Reference | GLOBAL | 2026-09-03 | 21 | `v2.product.update_item` |
| SPD-013 | unlist_item | `product/item/unlist-item.pdf` | API Reference | GLOBAL | 2026-09-03 | 22 | `v2.product.unlist_item` |
| SPD-014 | delete_item | `product/item/delete-item.pdf` | API Reference | GLOBAL | 2026-09-03 | 23 | `v2.product.delete_item` |
| SPD-015 | init_tier_variation | `product/variation-model/init-tier-variation.pdf` | API Reference | GLOBAL | 2026-09-01 | 06 | `v2.product.init_tier_variation` |
| SPD-016 | update_tier_variation | `product/variation-model/update-tier-variation.pdf` | API Reference | GLOBAL | 2026-09-03 | 17 | `v2.product.update_tier_variation` |
| SPD-017 | add_model | `product/variation-model/add-model.pdf` | API Reference | GLOBAL | 2026-09-01 | 07 | `v2.product.add_model` |
| SPD-018 | update_model | `product/variation-model/update-model.pdf` | API Reference | GLOBAL | 2026-09-03 | 18 | `v2.product.update_model` |
| SPD-019 | delete_model | `product/variation-model/delete-model.pdf` | API Reference | GLOBAL | 2026-09-03 | 19 | `v2.product.delete_model` |
| SPD-020 | get_model_list | `product/variation-model/get-model-list.pdf` | API Reference | GLOBAL | 2026-09-01 | 08 | `v2.product.get_model_list` |
| SPD-021 | get_category | `product/category/get-category.pdf` | API Reference | GLOBAL | 2026-09-01 | 09 | `v2.product.get_category` |
| SPD-022 | category_recommend | `product/category/category-recommend.pdf` | API Reference | GLOBAL | 2026-09-03 | 20 | `v2.product.category_recommend` |
| SPD-023 | get_attribute_tree | `product/attribute/get-attribute-tree.pdf` | API Reference | GLOBAL | 2026-09-01 | 10 | `v2.product.get_attribute_tree` |
| SPD-024 | get_recommend_attribute | `product/attribute/get-recommend-attribute.pdf` | API Reference | GLOBAL | 2026-09-01 | 11 | `v2.product.get_recommend_attribute` |
| SPD-025 | get_brand_list | `product/brand/get-brand-list.pdf` | API Reference | GLOBAL | 2026-09-01 | 12 | `v2.product.get_brand_list` |
| SPD-026 | register_brand | `product/brand/register-brand.pdf` | API Reference | GLOBAL | 2026-09-01 | 13 | `v2.product.register_brand` |
| SPD-027 | update_price | `product/pricing/update-price.pdf` | API Reference | GLOBAL | 2026-09-01 | 15 | `v2.product.update_price` |
| SPD-028 | update_stock | `product/inventory/update-stock.pdf` | API Reference | GLOBAL | 2026-09-01 | 16 | `v2.product.update_stock` |
| SPD-029 | get_item_limit | `product/limits/get-item-limit.pdf` | API Reference | GLOBAL | 2026-09-01 | 14 | `v2.product.get_item_limit` |
| SPD-030 | Push Notifications (Webhooks) | `push/push-notifications-webhooks.pdf` | Developer Guide | GLOBAL (pt-BR) | 2026-09-03 | — | webhook events / push notifications |
| SPD-031 | Sensitive Data | `policies/sensitive-data.pdf` | Developer Guide | GLOBAL (pt-BR) | 2026-09-03 | — | sensitive-data handling policy |
| SPD-032 | FAQ | `support/faq.pdf` | Developer Guide | GLOBAL (pt-BR) | 2026-09-03 | — | frequently asked questions |
| SPD-033 | Best practices before opening a ticket | `support/best-practices-before-opening-a-ticket.pdf` | Developer Guide | GLOBAL (pt-BR) | 2026-09-03 | — | support workflow |
| SPD-034 | References | `support/references.pdf` | Developer Guide | GLOBAL (pt-BR) | 2026-09-03 | — | reference links / announcements |
| SPD-035 | BR SPI App Creation User Guide | `brazil/br-spi-app-creation-user-guide.pdf` | Developer Guide | **BRAZIL** | 2026-09-03 | — | Brazil-specific SPI app creation |

Original page titles (Developer Guide, verbatim Portuguese) and full provenance:
`research/shopee-primary-docs/evidence-registry.md`.
