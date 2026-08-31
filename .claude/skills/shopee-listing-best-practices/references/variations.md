# Variations — Tier Variation & Model (combination / matrix)

last_reviewed: 2026-08-28
phase_02_2_reviewed: 2026-08-30
volatile: true
classification: OFFICIAL (pattern) — verification SEARCH_INDEXED; ALL numeric caps `UNVERIFIED` / `⚠ verify`
phase_02_2_note: >-
  Method names `init_tier_variation` (`item_id`, `tier_variation`, `model`),
  `add_model` (`item_id`, `model_list` → `model_id`s), `update_model`,
  `delete_model` (`item_id`, `model_id`), `get_model_list` (`item_id`),
  `get_variations` corroborated across two independent SDKs — `SEARCH_INDEXED` ·
  MEDIUM · MULTI_SOURCE. The Item→Model structure is a **corroborated API
  contract candidate** (usable for provisional mapping design, not an
  implementation contract). Models are created by a **separate call** after
  `add_item`. Whether a no-variation item still has a hidden default model is
  `UNRESOLVED`. All caps (tiers / options / models) still `UNVERIFIED`. Shopee
  also has a **kit / bundle** concept (`get_kit_limit`, `add_kit_item`, …) this
  file does not model. See `research/shopee-api-contract/phase-02.2-report.md` §10.

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

A no-variation item has no models; the sellable unit is the `item_id`.

## 2. Structure (all `SEARCH_INDEXED`, caps `⚠ verify`)

- **Tiers:** 0, 1, or 2. Cap of 2 corroborated across SDKs and BR integrator
  docs — **`⚠ verify` for BR.**
- **Options per tier:** an ordered list of option names; tier-1 options may each
  carry one image. v1 limited an option name to ~20 chars — **`UNVERIFIED` for
  v2 / BR.**
- **Model = combination.** `init_tier_variation` / `add_model` take a
  `model_list`; each model carries `tier_index`, `model_sku`, price, stock.
  Whether Shopee auto-creates the full matrix or only the models you submit is
  **`UNVERIFIED`** (v1 required each to be submitted — assume the same).
- **Post-sale mutability:** `init_tier_variation` reportedly "cannot edit an
  existing tier_variation"; structural change after sales is restricted.
  `update_model` / `update_tier_variation` handle price/stock/option edits within
  limits. Exact rules **`UNVERIFIED`.**
- **Option naming:** likely no special characters / no trailing spaces / not
  misleading (from BR title guidance) — **`UNVERIFIED`.**

## 3. Numeric limits — DO NOT LOCK

| Limit | Discovery value | Status |
|---|---|---|
| max tiers | 2 | `SEARCH_INDEXED`, MEDIUM — provisional, `⚠ verify` |
| max options per tier | — | `UNVERIFIED` |
| max models per item | SEA historically ~50 | `OTHER_MARKET_REFERENCE`, `⚠ verify` for BR |
| option-name length | ~20 chars (v1) | `UNVERIFIED` for v2 / BR |
| images per tier-1 option | 1 | `SEARCH_INDEXED`, `⚠ verify` |

All of these are `DYNAMIC` — resolve via `get_item_limit`
(`resolve_variation_limits`, `references/api-and-auth.md` §5). Anything the draft
needs beyond 2 tiers, or near any suspected cap, → flag for `REVIEW`.

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
- v2 model endpoint names (`init_tier_variation`, `add_model`, `update_model`,
  `get_model_list`) — `github.com/wjp-letgo/shopeego` — community SDK —
  consulted 2026-08-28 — `SEARCH_INDEXED`.
- 2-tier cap corroboration — BR integrator docs — external — consulted
  2026-08-28 — `SEARCH_INDEXED`, MEDIUM confidence.
- Full detail: `research/shopee-listing-skill/discovery-report.md` §6, §29 (U6).
