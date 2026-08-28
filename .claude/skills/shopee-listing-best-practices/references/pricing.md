# Pricing

last_reviewed: 2026-08-28
volatile: true
classification: OFFICIAL (fields) — verification SEARCH_INDEXED; all bounds `UNVERIFIED`

## 1. Conceptual separation (keep these four apart)

| Concept | What it is | Where it lives |
|---|---|---|
| **product fact** | intrinsic to the product? — **no.** Price is never `ProductMaster` truth. | not a product fact |
| **commercial context** | acquisition cost, target margin, competitor targets, promo strategy | `COMMERCIAL_OPTIONAL` (SKILL.md §3); missing → WARNING, analysis unavailable |
| **publication requirement** | a price must be present and within the category's bounds to publish | `PUBLICATION_REQUIRED`; unresolved → `REVIEW` |
| **execution mechanism** | the `UPDATE_PRICE` operation and its prerequisites | `EXECUTION` (per operation) |

## 2. What discovery found (all `SEARCH_INDEXED`, `⚠ verify`)

| Aspect | Finding | Status |
|---|---|---|
| Level | model-level when models exist, else item-level | `SEARCH_INDEXED` |
| Fields | `original_price` and a promo / `current_price`; currency **BRL** | `SEARCH_INDEXED` |
| Update | `update_price` (+ batch); `update_model` for a model's price | `SEARCH_INDEXED` |
| Bounds | `get_item_limit.price_limit` (min / max) | values `UNVERIFIED` — `DYNAMIC` |
| Range display | the item shows a price range when models differ | `SEARCH_INDEXED` |
| Misleading pricing | Shopee BR polices fake "de / por" reference prices | `SEARCH_INDEXED`; specifics `UNVERIFIED` |
| Cross-variant consistency | no rule found that all models must share a price; kit / pack differences allowed | `UNVERIFIED` |

## 3. Do NOT hardcode

- minimum / maximum price,
- variation (model-to-model) price gap limits,
- discount / promotional constraints,
- promo mechanics.

All remain unresolved. Resolve `price_limit` via `get_item_limit`
(`resolve_price_bounds`, `references/api-and-auth.md` §5).

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
