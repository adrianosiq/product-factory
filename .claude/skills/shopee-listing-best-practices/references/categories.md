# Categories — discover → resolve → validate

last_reviewed: 2026-08-28
phase_02_3_reviewed: 2026-09-03
phase_02_2_reviewed: 2026-08-30
volatile: true
classification: workflow INTERNAL; underlying endpoints OFFICIAL (verification PRIMARY_VERIFIED — SPD-021/022)
phase_02_3_note: >-
  PRIMARY (SPD-021 get_category, SPD-022 category_recommend, SPD-010).
  `get_category` (`GET`, param `language` only — **returns the whole tree**, no
  `category_id`) → `category_list[{category_id, parent_category_id,
  original_category_name, display_category_name, has_children}]`. **Leaf =
  `has_children: false`** (PRIMARY_VERIFIED mechanism; an explicit "must list
  under a leaf" sentence is `PRIMARY_PARTIAL`). No `listing_allowed` flag —
  restriction surfaces at `add_item` as `error_category_is_block` "Category is
  restricted" / `error_forbidden_category` / `error_invalid_category` "L1 and L2
  do not match". `category_recommend` (`GET`, `item_name` + optional
  `product_cover_image` id) → ranked `category_id[]` + a `ds_cat_rcmd_id` for
  `add_item`.
phase_02_2_note: >-
  HISTORICAL (superseded by phase_02_3_note above). `product/get_category` and
  `product/category_recommend` corroborated across two SDKs — then
  `SEARCH_INDEXED`. Phase 02.3 (SPD-021) verified the tree + `has_children`
  mechanism: **leaf = `has_children: false`** is `PRIMARY_VERIFIED`; an explicit
  "must list under a leaf" sentence is `PRIMARY_PARTIAL` (implied by per-leaf
  brand/attribute + `error_invalid_category`). No `listing_allowed`-style flag.
  See `research/shopee-api-contract/phase-02.2-report.md` §11.

## 1. Safe workflow (INTERNAL)

Same discipline as the Mercado Livre Skill — a predictor is **discovery, not
authority**:

```
discover   candidate categories (predictor, if it exists; or an internal mapping)
   ↓
resolve    pick a candidate LEAF; check it fits the real product
   ↓
validate   confirm leaf; pull the attribute set, brand list and DTS limits for it
```

Do **not** claim Shopee exposes a category predictor unless verified. If
`category_recommend` is unavailable, discovery falls back to an internal
category mapping that is itself revalidated — never skip `validate`.

## 2. What is known (Phase 02.3 primary — SPD-021, SPD-022)

| Aspect | Finding | Tag | Verification |
|---|---|---|---|
| Tree | `get_category` (`GET`, param `language` only — returns the **whole tree**, no `category_id`) → `category_id`, `parent_category_id`, `original_category_name`, `display_category_name`, `has_children` | OFFICIAL | `PRIMARY_VERIFIED` (SPD-021) |
| Leaf **identification** | leaf = `has_children: false` | OFFICIAL | `PRIMARY_VERIFIED` mechanism (SPD-021) |
| Leaf **requirement** | that `add_item` must sit under a leaf — no explicit sentence in the corpus; implied by per-leaf brand/attribute lookups + `error_invalid_category` "L1 and L2 do not match" | OFFICIAL | `PRIMARY_PARTIAL` — do not overstate as a proven universal rule |
| Prediction | `category_recommend` (`GET`, `item_name` + optional `product_cover_image` id) → ranked `category_id[]` + a `ds_cat_rcmd_id` for `add_item` | OFFICIAL | `PRIMARY_VERIFIED` (SPD-022) |
| Region scope | tree is **per region** (a BR tree); `language` param; not shop-specific | OFFICIAL | `PRIMARY_VERIFIED` (SPD-021) |
| Category drives | mandatory attributes; whether brand is required; days-to-ship limits; size-chart support; allowed logistics; allowed `condition`; prohibited / restricted status | OFFICIAL | mixed — see the specific reference file per driver |
| No `listing_allowed` flag | restriction surfaces at `add_item` as `error_category_is_block` / `error_forbidden_category` / `error_invalid_category` | OFFICIAL | `PRIMARY_VERIFIED` (SPD-021, SPD-010) |
| Change after publish | possible via `update_item`; not documented in the corpus | OFFICIAL | `PRIMARY_NOT_FOUND` |

## 3. Open questions (resolve before enforcing)

- **Category-tree endpoint** shape — **resolved** (SPD-021: `GET`, `language`
  only, whole tree, `has_children`).
- **Leaf requirement** — leaf *identification* is `PRIMARY_VERIFIED`
  (`has_children: false`); that `add_item` *universally requires* a leaf is
  `PRIMARY_PARTIAL` (implied, not stated). Do not turn the mechanism into a
  broader publication rule beyond what the corpus supports.
- **Active / inactive categories** — no `status` / `listing_allowed` field;
  restriction surfaces as `add_item` errors (`PRIMARY_VERIFIED`).
- **Seller / category restrictions** — which categories are gated by
  authorisation, brand IP, or shop type — explicit list `PRIMARY_NOT_FOUND`.
- **Category migration** — behaviour when changing a live listing's category —
  `PRIMARY_NOT_FOUND`.
- **Category-specific limits** — title length, image count, DTS, variation caps
  are per-category via **`get_item_limit`** (`PRIMARY_VERIFIED` source; `add_item`
  prose also names a dedicated `get_dts_limit` page, not in the corpus);
  **never hardcode** them (see `references/api-and-auth.md` §0 / §5).

## 4. Readiness impact

- Category not yet resolved/validated (data pending) → `PUBLICATION_STATUS =
  REVIEW`, `dynamic_checks_required: resolve_leaf_category`.
- Validation executed and the category is not a leaf / not usable / wrong for the
  product → `PUBLICATION_STATUS = FAIL`.
- A wrong or unusable category does **not** by itself fail `CONTENT_STATUS` —
  product truth can still be sufficient.
- `CONFLICTING` category signals (e.g. predictor vs internal mapping disagree on
  a materially different product) → human review, never auto-resolved.

## Sources

- **PRIMARY** — `docs/marketplaces/shopee/open-platform/product/category/get-category.pdf`
  (`SPD-021`) + `category-recommend.pdf` (`SPD-022`) — Shopee Open Platform API
  Reference — read 2026-09-03 — `PRIMARY_VERIFIED`: `get_category` (`language`
  only, whole tree, `has_children`); `category_recommend` (`item_name` +
  optional `product_cover_image` → ranked `category_id[]` + `ds_cat_rcmd_id`);
  no `listing_allowed` flag. Registry: `research/shopee-primary-docs/`.
- `get_category` / `category_recommend` names & fields — `github.com/wjp-letgo/shopeego`
  — community SDK — consulted 2026-08-28 — `SEARCH_INDEXED` (superseded by SPD-021/022).
- Category drives attributes / brand / DTS / restrictions — `seller.shopee.com.br/edu`,
  `help.shopee.com.br` — Centro de Educação / Central de Ajuda — consulted
  2026-08-28 — `SEARCH_INDEXED` (snippets only).
- Discover → resolve → validate discipline — `.claude/skills/mercado-livre-listing-best-practices/references/categories.md`
  — internal — architectural reference only.
- Full detail: `research/shopee-listing-skill/discovery-report.md` §4.
