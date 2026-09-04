# Pricing

last_reviewed: 2026-08-28
phase_02_3_reviewed: 2026-09-03
phase_02_2_reviewed: 2026-08-30
volatile: true
classification: OFFICIAL (fields) — `update_price` + `price_limit` source PRIMARY_VERIFIED (SPD-027/029); the bound values are DYNAMIC
phase_02_3_note: >-
  PRIMARY (SPD-027 update_price, SPD-029, SPD-011). `POST /api/v2/product/update_price`;
  `item_id` + `price_list[]` (**length 1–50**) `{model_id (0 = no model item),
  original_price float}`. **Price bounds are `get_item_limit.price_limit
  {min_limit, max_limit}`** (per shop+category, DYNAMIC) — `error_price_exceed_min_limit`
  / `error_price_exceed_max_limit` / `error_price_out_of_range`. **BR (+ SG/MY/
  MX/PL/ES/AR): two decimal places allowed**; other regions integer only.
  `original_price` vs `current_price` (= promo price when `has_promotion`);
  `get_item_base_info` omits `price_info` when the item has models (use
  `get_model_list`). Can't edit item price directly when it has models
  (`error_edit_item_price_for_item_has_model`); locked under promotion;
  `error_seller_under_penalty` blocks edits.
phase_02_2_note: >-
  HISTORICAL (superseded by phase_02_3_note above). Phase 02.2 left the
  `price_limit` resolution source `UNVERIFIED` and the `get_item_limit` scope
  disputed. Phase 02.3 (SPD-029) confirmed **`get_item_limit.price_limit
  {min_limit, max_limit}` IS the source** (per shop+category, DYNAMIC values).
  See `phase-02.2-report.md` §18.

## 1. Conceptual separation (keep these four apart)

| Concept | What it is | Where it lives |
|---|---|---|
| **product fact** | intrinsic to the product? — **no.** Price is never `ProductMaster` truth. | not a product fact |
| **commercial context** | acquisition cost, target margin, competitor targets, promo strategy | `COMMERCIAL_OPTIONAL` (SKILL.md §3); missing → WARNING, analysis unavailable |
| **publication requirement** | a price must be present and within the category's bounds to publish | `PUBLICATION_REQUIRED`; unresolved → `REVIEW` |
| **execution mechanism** | the `UPDATE_PRICE` operation and its prerequisites | `EXECUTION` (per operation) |

## 2. What is known (Phase 02.3 primary — SPD-027, SPD-029)

| Aspect | Finding | Status |
|---|---|---|
| Level | model-level when models exist (`price_list` entry per `model_id`; `0` = no-model); item-level price blocked when the item has models (`error_edit_item_price_for_item_has_model`) | `PRIMARY_VERIFIED` (SPD-027) |
| Fields | `original_price` (float) and `current_price` (= promo price when `has_promotion`); BR allows **2 decimal places** | `PRIMARY_VERIFIED` (SPD-027/011) |
| Update | `POST /api/v2/product/update_price`, `price_list[]` 1–50 | `PRIMARY_VERIFIED` (SPD-027) |
| Bounds | **`get_item_limit.price_limit {min_limit, max_limit}`** — per shop+category | `PRIMARY_VERIFIED` source (SPD-029); values `DYNAMIC` |
| Range display | the item shows a price range when models differ; `get_item_base_info` omits `price_info` for a multi-model item (use `get_model_list`) | `PRIMARY_VERIFIED` (SPD-011) |
| Misleading pricing | Shopee BR polices fake "de / por" reference prices | `SEARCH_INDEXED`; specifics `UNVERIFIED` |
| Cross-variant consistency | no rule found that all models must share a price | `PRIMARY_NOT_FOUND` |

## 3. Do NOT hardcode

- minimum / maximum price,
- variation (model-to-model) price gap limits,
- discount / promotional constraints,
- promo mechanics.

The **values** stay `DYNAMIC` per shop+category. Resolve `price_limit` via
`get_item_limit` (`resolve_price_bounds`, `references/api-and-auth.md` §0 / §5) —
never assume a fixed bound.

## 4. Readiness impact

- Price absent on the draft → `PUBLICATION_STATUS = REVIEW`
  (`PUBLICATION_REQUIRED` gap), not a content blocker.
- `price_limit` unresolved → `PUBLICATION_STATUS = REVIEW`
  (`resolve_price_bounds`).
- Resolved and price outside `price_limit` → `PUBLICATION_STATUS = FAIL`.
- A fake reference / "de por" price the evidence does not support → compliance
  finding (`references/compliance.md`); at least `MAJOR`.
- Missing acquisition cost / margin data → WARNING ("pricing / profitability
  analysis unavailable"), never `BLOCKER`.

## Sources

- Model/item price level, `original_price` / promo price, BRL, `update_price`,
  price range display — `github.com/wjp-letgo/shopeego`, `github.com/teacat/shopeego`
  — community SDKs — consulted 2026-08-28 — `SEARCH_INDEXED`.
- `get_item_limit.price_limit` — `github.com/wjp-letgo/shopeego` — community SDK
  — consulted 2026-08-28 — `SEARCH_INDEXED`; values `UNVERIFIED`.
- Misleading-price policing — BR educators — external — consulted 2026-08-28 —
  `SEARCH_INDEXED`.
- Full detail: `research/shopee-listing-skill/discovery-report.md` §13.
