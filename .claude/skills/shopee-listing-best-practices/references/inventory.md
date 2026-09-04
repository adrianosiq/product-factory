# Inventory — CONSERVATIVE (major discovery gap)

last_reviewed: 2026-08-28
phase_02_3_reviewed: 2026-09-03
phase_02_2_reviewed: 2026-08-30
volatile: true
classification: OFFICIAL (level) — verification SEARCH_INDEXED; the BR stock/warehouse model is UNRESOLVED
phase_02_3_note: >-
  PRIMARY (SPD-028 update_stock, SPD-015, SPD-011, SPD-020). `POST /api/v2/product/update_stock`;
  one `item_id` per call; `stock_list[]` (**length 1–50**) `{model_id (0 = no
  model item), seller_stock:[{location_id (optional), stock}]}`. **Writes only
  `seller_stock`; the value is ABSOLUTE** ("new stock info"); `normal_stock` was
  offlined 2022-10-31. **Multi-warehouse IS supported**: `location_id` comes from
  **`v2.shop.get_warehouse_detail`** (page NOT in corpus); omit it when the shop
  has no warehouse; **cannot mix** stock structures (all with `location_id` or
  all without). WMS shops blocked (`error_wms_shop_block_upate_stock`); FBS/B2C:
  normal stock must = 0. Reads: `stock_info_v2 {summary_info{total_reserved_stock,
  total_available_stock}, seller_stock[{location_id, stock, if_saleable}],
  shopee_stock[...], advance_stock{...}}`. Do **not** import Multi Origem — this
  is Shopee's own model.
phase_02_2_note: >-
  `product/update_stock` (`item_id`, `stock_list`) corroborated; `stock_list`
  entries carry `model_id` when models exist. No location/warehouse dimension was
  observed in triangulation — but absence in a summary is not proof. State stays
  "inventory object partially verified; warehouse topology UNRESOLVED". See
  `phase-02.2-report.md` §19–§20.

## 1. Why this file stays minimal

Inventory is one of the largest Phase 01 gaps (SKILL.md gap G4). This file
intentionally states little and locks nothing.

## 2. What is (weakly) known

| Aspect | Finding | Status |
|---|---|---|
| Level | stock appears **model-level** where variations / models exist; item-level otherwise | `SEARCH_INDEXED`, MEDIUM |
| Update | `update_stock` (+ batch); `update_model` for a model's stock | `SEARCH_INDEXED` |
| Reserved vs available | Shopee tracks reserved vs sellable internally; the seller writes the sellable figure | `SEARCH_INDEXED` |
| Fulfilment stock | inventory in Shopee's DCs / "Envios Shopee" is **not** freely seller-writable via `update_stock` | `SEARCH_INDEXED` |

## 3. What is UNVERIFIED (do not assume)

- the exact **stock endpoint** shape (`stock_info_v2` / `seller_stock` /
  `shop_stock` — names only),
- **warehouse semantics** — whether BR uses a single seller pool or a
  multi-warehouse `location_id` dimension,
- **available vs reserved** field names / structure,
- **multi-warehouse support** for BR at all,
- **absolute vs incremental** update semantics for `update_stock`,
- **concurrency behaviour** — no optimistic-lock / version token found;
  last-write-wins assumed but unconfirmed.

## 4. Phase-02 safe default

Assume **one seller-managed stock figure per model** (or per item when there are
no models) in BR, set as an **absolute** value. Treat multi-warehouse as a
`DYNAMIC` account / region capability check (`resolve_inventory_model`,
`references/api-and-auth.md` §5), not a default.

## 5. Do NOT import Multi Origem

> **Shopee inventory architecture must be discovered independently.**

Do **not** create for Shopee: `STANDARD`, `MULTI_ORIGIN_SINGLE_WAREHOUSE`,
`MULTI_ORIGIN_MULTIWAREHOUSE`, `warehouse_management` tags, `stock_locations`,
`network_node_id`, `selling_address` / `seller_warehouse` stock types. Those are
Mercado Livre concepts.

## 6. Readiness impact

- Stock absent on the draft → `PUBLICATION_STATUS = REVIEW`
  (`PUBLICATION_REQUIRED` gap).
- BR inventory model unresolved → `EXECUTION_STATUS = REVIEW` for any
  `UPDATE_STOCK` operation (`resolve_inventory_model`) — **never `FAIL`** merely
  because it is unresolved.
- An executed operation that writes a stock shape BR rejects → `EXECUTION_STATUS
  = FAIL`.
- Stock is never `CONTENT_STATUS` — it does not affect product truth.

## Sources

- Model/item stock level, `update_stock` / `update_model`, fulfilment stock not
  seller-writable — `github.com/wjp-letgo/shopeego`, `rollout.com`,
  `api2cart.com` — community SDK / external — consulted 2026-08-28 —
  `SEARCH_INDEXED`.
- `stock_info_v2` / `seller_stock` / `location_id` in some markets — SEA docs —
  consulted 2026-08-28 — `OTHER_MARKET_REFERENCE` / `UNVERIFIED` for BR.
- Multi Origem non-mapping — `.claude/skills/mercado-livre-listing-best-practices/references/variations-and-user-products.md`
  — internal — architectural reference only.
- Full detail: `research/shopee-listing-skill/discovery-report.md` §14, §15,
  §29 (U7).
