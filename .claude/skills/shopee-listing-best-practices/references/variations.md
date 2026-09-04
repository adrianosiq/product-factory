# Variations — Tier Variation & Model (combination / matrix)

last_reviewed: 2026-08-28
phase_02_3_reviewed: 2026-09-03
phase_02_2_reviewed: 2026-08-30
volatile: true
classification: OFFICIAL (pattern) — verification PRIMARY_VERIFIED (SPD-015); model-count is PRIMARY_CONFLICT; name-length caps are DYNAMIC
phase_02_3_note: >-
  PRIMARY (SPD-015 init_tier_variation, SPD-017 add_model, SPD-020 get_model_list,
  SPD-029). **Max 2 tiers — PRIMARY_VERIFIED** (`error_param` "The level of
  tier-variation over 2"). The old `tier_variation` request structure is
  **deprecated (2025-09-12)** → use `standardise_tier_variation`
  (`variation_id`, `variation_name`, `variation_group_id`,
  `variation_option_list[{variation_option_id, variation_option_name, image_id}]`).
  **`model_id = 0` = "no model item"**; Shopee keeps an internal **default
  model** for no-variation items. Counts: model list per call ≤ 50; total models
  per item `< 20 (50 for TW)` (PRIMARY_CONFLICT with "at most 50"); options per
  tier ≤ 20; combinations ≤ 50; `model_sku` ≤ 100 chars; tier/option **name**
  length from `get_item_limit`. `model_status` ∈ `MODEL_NORMAL` /
  `MODEL_UNAVAILABLE` (whitelist to set). Structure change LOCKED under
  promotion. FBS item/model not editable; CNSC shop cannot change tier structure.
phase_02_2_note: >-
  HISTORICAL (superseded by phase_02_3_note above). Method names
  `init_tier_variation` / `add_model` / `update_model` / `delete_model` /
  `get_model_list` were corroborated across two SDKs — then `SEARCH_INDEXED`.
  Phase 02.3 (SPD-015/017/020) made the Item→Model structure `PRIMARY_VERIFIED`,
  fixed **max 2 tiers** (`PRIMARY_VERIFIED`), the `model_id = 0` no-model
  convention, and the model-count `PRIMARY_CONFLICT`. Shopee also has a **kit /
  bundle** concept (`get_kit_limit`, `add_kit_item`, …) this file does not model.
  See `research/shopee-api-contract/phase-02.2-report.md` §10.

## 1. The combination pattern (maps well to Product Factory)

Shopee appears to use a **combination / matrix** model — strong Phase 01
evidence:

```
Tier 1  Cor:     Preto, Branco
Tier 2  Tamanho: P, M, G

→ 6 sellable Models, each = one (Cor, Tamanho) combination
   model.tier_index = [0,0] [0,1] [0,2] [1,0] [1,1] [1,2]
```

Each model has its own `model_id` (Shopee-assigned), `model_sku`
(seller-controlled), price and stock. This maps cleanly to internal Product
Factory variants **at the model level**: `variant_id ↔ model_id`, internal SKU
written into `model_sku`. The `tier_index` + option names are the human-readable
axis values — **not** identity.

For a no-variation item, the `model_id = 0` addressing convention applies to
per-model price/stock operations (`PRIMARY_VERIFIED`, SPD-016/027); whether a
fully queryable default-model entity exists for every such item across every API
is inferred from one FBS-context phrase and is **`PRIMARY_PARTIAL`** — Product
Factory maps `variant_id → (item_id, model_id = 0)` and keeps its own
sellable-unit identity either way.

## 2. Structure (Phase 02.3 primary — SPD-015, SPD-017, SPD-020, SPD-029)

- **Tiers:** 0, 1, or 2. **Max 2 tiers is `PRIMARY_VERIFIED`** (`error_param`
  "The level of tier-variation over 2"). The old `tier_variation` request
  structure is **deprecated (2025-09-12)** → use `standardise_tier_variation`.
- **Options per tier:** an ordered list of option names; tier-1 options may each
  carry one image. Option **count** per tier ≤ 20 (`error_tier_opt_too_many`
  "larger than 20"); a separate `error_param` also states options "should be
  under 50" — a **mild 20-vs-50 wording tension**, treat 20 as the working cap
  and flag near it. Option/tier **name length** comes from `get_item_limit`
  (`tier_variation_name_length_limit` / `tier_variation_option_length_limit`).
- **Model = combination.** `init_tier_variation` / `add_model` take a
  `model_list`; each model carries `tier_index`, `model_sku`, price, stock.
- **Post-sale mutability:** structure change (0↔1↔2 tiers) is supported but
  **LOCKED while the item is under promotion** (`error_cannt_*_in_promotion`);
  CNSC shops blocked; FBS item/model not editable (SPD-015, `PRIMARY_VERIFIED`).
- **Option naming:** no special characters / no trailing spaces / not
  misleading (from BR title guidance) — `SEARCH_INDEXED`.

## 3. Numeric limits

| Limit | Value | Status |
|---|---|---|
| max tiers | **2** | `PRIMARY_VERIFIED` (SPD-015) — a static rule |
| max options per tier | **≤ 20** (with a 20-vs-50 wording tension, see §2) | `PRIMARY_PARTIAL` (SPD-015) |
| max models per item | **`< 20` (50 for TW)** per an error, vs request text "at most 50" | **`PRIMARY_CONFLICT`** (SPD-015) — do **not** pick a side; flag for `REVIEW` and Phase 02.4 |
| model list per call | ≤ **50** (`init_tier_variation` / `add_model`) | `PRIMARY_VERIFIED` (SPD-015) |
| combinations | ≤ **50** | `PRIMARY_PARTIAL` (SPD-015) |
| `model_sku` length | ≤ **100** chars | `PRIMARY_VERIFIED` (SPD-015) |
| tier / option **name** length | via `get_item_limit` | `PRIMARY_VERIFIED` source (SPD-029) |
| images per tier-1 option | 1 | `PRIMARY_VERIFIED` (SPD-015) |

Max 2 tiers is a **static** primary rule. The name-length caps are `DYNAMIC` —
resolve via `get_item_limit` (`resolve_variation_limits`,
`references/api-and-auth.md` §5). Anything the draft needs beyond 2 tiers, or
near the model-count conflict, → flag for `REVIEW`.

## 4. Readiness impact

- Variation caps not resolved → `PUBLICATION_STATUS = REVIEW`,
  `dynamic_checks_required: resolve_variation_limits`.
- Resolved cap exceeded (e.g. 3 tiers, or models over the confirmed max) →
  `PUBLICATION_STATUS = FAIL`.
- A model whose identity (which real variant it is) is unestablished →
  `CONTENT_STATUS = FAIL` (a core variant identity is CORE_REQUIRED).
- Ambiguous option names or a model image that doesn't match the variant →
  return-prevention hotspot (`references/return-prevention.md`); at least
  `MAJOR`.

## 5. Not imported from Mercado Livre

No `PARENT_PK` / `CHILD_PK` (tiers are positional, seller-declared, not derived
from attribute metadata). No `Family`. No `User Product`. No `Multi Origem`.
Stock is per model — see `references/inventory.md`.

## Sources

- `tier_index` combination semantics, option-name ~20 chars, "cannot edit
  existing tier_variation" — `github.com/teacat/shopeego` (v1) — community SDK —
  consulted 2026-08-28 — `SEARCH_INDEXED` (v1; numbers stale).
- **PRIMARY** — `docs/marketplaces/shopee/open-platform/product/variation-model/`
  `init-tier-variation.pdf` (`SPD-015`), `add-model.pdf` (`SPD-017`),
  `get-model-list.pdf` (`SPD-020`) + `get-item-limit.pdf` (`SPD-029`) — Shopee
  Open Platform API Reference — read 2026-09-03 — `PRIMARY_VERIFIED`: **max 2
  tiers**, `standardise_tier_variation` (old structure deprecated 2025-09-12),
  `model_id = 0` = no-model, `model_sku` ≤ 100, model list ≤ 50 per call,
  model-count `PRIMARY_CONFLICT` (`< 20` vs "at most 50"). Registry:
  `research/shopee-primary-docs/`.
- v2 model endpoint names — `github.com/wjp-letgo/shopeego` — community SDK —
  consulted 2026-08-28 — `SEARCH_INDEXED` (superseded by SPD-015).
- 2-tier cap corroboration — BR integrator docs — external — consulted
  2026-08-28 — `SEARCH_INDEXED` (now `PRIMARY_VERIFIED` via SPD-015).
- Full detail: `research/shopee-listing-skill/discovery-report.md` §6, §29 (U6).
