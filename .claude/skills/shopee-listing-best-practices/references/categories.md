# Categories — discover → resolve → validate

last_reviewed: 2026-08-28
phase_02_2_reviewed: 2026-08-30
volatile: true
classification: workflow INTERNAL; underlying endpoints OFFICIAL (verification SEARCH_INDEXED, `⚠ verify`)
phase_02_2_note: >-
  `product/get_category` (param `language`) and `product/category_recommend`
  (param `item_name`) corroborated across two independent SDKs — `SEARCH_INDEXED`,
  MEDIUM. The **leaf-only requirement is still `UNRESOLVED`** (likely, not
  proven — do not state as OFFICIAL). No `listing_allowed`-style flag seen. See
  `research/shopee-api-contract/phase-02.2-report.md` §11.

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

## 2. What discovery found (all `SEARCH_INDEXED`, `⚠ verify`)

| Aspect | Finding | Tag | Verification |
|---|---|---|---|
| Tree | `get_category` → `category_id`, `parent_category_id`, `original_category_name`, `display_category_name`, `has_children` | OFFICIAL | `SEARCH_INDEXED` |
| Leaf required | list under a leaf (`has_children = false`) | OFFICIAL | `SEARCH_INDEXED` (strongly implied) |
| Prediction | `category_recommend` → ranked `category_id` list from item name / image | OFFICIAL | `SEARCH_INDEXED` |
| Region scope | tree is **per region** (a BR tree); a `language` / region param is passed; not shop-specific | OFFICIAL | `SEARCH_INDEXED` |
| Category drives | mandatory attributes; whether brand is required; days-to-ship limits; size-chart support; allowed logistics; allowed `condition`; prohibited / restricted status | OFFICIAL | `SEARCH_INDEXED` |
| Change after publish | possible via `update_item`; may invalidate attributes; some categories restricted | OFFICIAL | `UNVERIFIED` |

## 3. Open questions (resolve before enforcing)

- **Category-tree endpoint** shape and paging — `⚠ verify`.
- **Leaf-only requirement** — strongly implied, not confirmed.
- **Active / inactive categories** — is there an explicit eligibility / `status`
  field on a category? (No `listing_allowed` analogue found.) `UNVERIFIED`.
- **Seller / category restrictions** — which categories are gated by
  authorisation, brand IP, or shop type. `UNVERIFIED`.
- **Category migration** — behaviour and constraints when changing a live
  listing's category. `UNVERIFIED`.
- **Category-specific limits** — title length, image count, DTS, variation caps
  are believed to be per-category via `get_item_limit` / `get_dts_limit`;
  **never hardcode** them (see `references/api-and-auth.md` §5).

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

- `get_category` / `category_recommend` names & fields — `github.com/wjp-letgo/shopeego`
  — community SDK — consulted 2026-08-28 — `SEARCH_INDEXED`.
- Category drives attributes / brand / DTS / restrictions — `seller.shopee.com.br/edu`,
  `help.shopee.com.br` — Centro de Educação / Central de Ajuda — consulted
  2026-08-28 — `SEARCH_INDEXED` (snippets only).
- Discover → resolve → validate discipline — `.claude/skills/mercado-livre-listing-best-practices/references/categories.md`
  — internal — architectural reference only.
- Full detail: `research/shopee-listing-skill/discovery-report.md` §4.
