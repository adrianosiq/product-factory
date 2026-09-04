# Moderation & enforcement — post-publication, separate from ProductMaster truth

last_reviewed: 2026-08-28
phase_02_3_reviewed: 2026-09-03
phase_02_2_reviewed: 2026-08-30
volatile: true
classification: OFFICIAL (existence) — verification SEARCH_INDEXED; mechanics & API `⚠ verify` / `UNVERIFIED`
phase_02_3_note: >-
  PRIMARY (SPD-011 get_item_base_info). **`item_status` enum (definitive):
  `NORMAL`, `BANNED`, `UNLIST`, `SELLER_DELETE`, `SHOPEE_DELETE`, `REVIEWING`**
  — CORRECTS the `DELETED` guess. `get_item_base_info` also returns `deboost`
  (bool — "search ranking is lowered"). `error_seller_under_penalty` ("The shop
  is under penalty") blocks price/stock edits (SPD-027/028). **`get_item_violation_info`
  and content-diagnosis endpoints are NOT in the 35-PDF primary corpus** — the
  Phase 02.2 SEARCH_INDEXED names remain unverified; do not rely on them. No
  penalty-points API page was supplied. Enforcement state stays separate from
  ProductMaster truth (unchanged).
phase_02_2_note: >-
  HISTORICAL (superseded by phase_02_3_note above). Phase 02.2 surfaced
  community-SDK names `get_item_violation_info`, `get_item_content_diagnosis_result`,
  `get_item_list_by_content_diagnosis` — **none are in the 35-PDF primary corpus**
  (`PRIMARY_NOT_FOUND`); `NOT PRIMARY VERIFIED — DO NOT RELY ON FOR EXECUTION`.
  The `item_status` enum IS now re-verified (SPD-011): see phase_02_3_note. See
  `research/shopee-api-contract/phase-02.2-report.md` §23–§24.

## 1. Why this file is separate

Phase 01 found seller / listing enforcement to be prominent on Shopee BR
(penalty points, bans). It must stay **separate from pre-publication readiness**
and **separate from `ProductMaster` truth**. Enforcement state is an `EXECUTION`
/ compliance input, never a `CONTENT` input.

## 2. Listing status graph

`item_status` (v2) — **`PRIMARY_VERIFIED`** enum (SPD-011):

| State | Meaning | Verification |
|---|---|---|
| `NORMAL` | live and for sale | `PRIMARY_VERIFIED` |
| `UNLIST` | exists but hidden / not for sale (seller toggled) | `PRIMARY_VERIFIED` |
| `BANNED` | removed by Shopee for a policy violation | `PRIMARY_VERIFIED` |
| `SELLER_DELETE` | deleted by the seller | `PRIMARY_VERIFIED` |
| `SHOPEE_DELETE` | deleted by Shopee | `PRIMARY_VERIFIED` |
| `REVIEWING` | pending moderation after create / edit | `PRIMARY_VERIFIED` |

There is **no** plain `DELETED` — seller vs Shopee deletion are distinct states.
`get_item_base_info` also returns `deboost` (bool — "search ranking is lowered").
State model: `NOT_CREATED → (create) → [REVIEWING] → NORMAL ⇄ UNLIST`, with
`BANNED` / `SELLER_DELETE` / `SHOPEE_DELETE` terminal-ish. Whether an edit
re-triggers `REVIEWING` is `PRIMARY_NOT_FOUND`. Do not invent transitions from
the enum's existence.

## 3. Penalty-point system (BR) — conceptual conclusion only

**Do not hardcode** the weekly cadence, the 60-day validity, or progressive
threshold values. What we keep:

> Seller / listing enforcement exists on Shopee BR and accrues against the shop.
> It must remain separate from `ProductMaster` truth.

Reconstructed mechanics (`SEARCH_INDEXED`, `⚠ verify`, SKILL.md gap G7): points
accrue ~weekly, valid ~60 days; each infraction has a point value (e.g.
incomplete listing ≈ 3); thresholds trigger progressive sanctions — search
deprioritisation → feature limits → listing-count limits → account suspension.
An appeals / correction flow exists via Seller Center.

## 4. API exposure gaps

- `item_status = BANNED` and `deboost` are visible via `get_item_base_info` /
  `get_item_list` (`PRIMARY_VERIFIED`).
- Community-SDK names `product/get_item_violation_info`,
  `product/get_item_content_diagnosis_result`,
  `product/get_item_list_by_content_diagnosis` are **`NOT PRIMARY VERIFIED — DO
  NOT RELY ON FOR EXECUTION`**: none appear in the 35-PDF primary corpus
  (`PRIMARY_NOT_FOUND`). This does **not** assert they don't exist — only that
  the corpus does not document them. Use `item_status` + `deboost` in the
  meantime.
- A shop-level penalty / account-health API for **BR** is still `UNVERIFIED`
  (other Shopee markets expose `public/get_shop_penalty` / account-health — BR
  availability unconfirmed). The BR `br` service exposes `query_shop_block_status`
  / `query_sku_block_status` (`SEARCH_INDEXED`, S12).
- Much enforcement state is likely **Seller-Center-only** — treat it as
  requiring human input.

## 5. Readiness impact

- Post-publication only: an unchecked moderation / penalty state →
  `QUALITY_STATUS = REVIEW` until read; a confirmed adverse state → route to a
  `compliance_finding` and `EXECUTION_STATUS`.
- `item_status ∈ {BANNED, SELLER_DELETE, SHOPEE_DELETE}` for an update operation
  → `EXECUTION_STATUS = FAIL` for that operation.
- Never recreate a listing to dodge a ban. Preserve `reason` + `remedy`; never
  fabricate a remedy.
- "No moderation found" is **not** proof a payload is compliant.

## Sources

- **PRIMARY** — `docs/marketplaces/shopee/open-platform/product/item/get-item-base-info.pdf`
  (`SPD-011`) — Shopee Open Platform API Reference — read 2026-09-03 —
  `PRIMARY_VERIFIED`: `item_status ∈ {NORMAL, BANNED, UNLIST, SELLER_DELETE,
  SHOPEE_DELETE, REVIEWING}` (no plain `DELETED`); `deboost` bool. Registry:
  `research/shopee-primary-docs/`.
- Earlier `item_status` guess (incl. plain `DELETED`) —
  `github.com/wjp-letgo/shopeego` — community SDK — consulted 2026-08-28 —
  `SEARCH_INDEXED` (**corrected by SPD-011**).
- Per-item violation / content-diagnosis method names — community SDKs —
  consulted 2026-08-28 — `NOT PRIMARY VERIFIED` (not in the 35-PDF corpus).
- Penalty-point mechanics — "Pontos de Penalidade" Shopee BR PDF (S5), BR
  educators (`gobots`, `destraveescale`, `1001clicks`, `mambadigital`) — Shopee
  BR / external — consulted 2026-08-28 — `SEARCH_INDEXED` (PDF referenced, not
  parsed); numbers `⚠ verify`.
- `public/get_shop_penalty` — other Shopee markets — consulted 2026-08-28 —
  `UNVERIFIED` for BR.
- Enforcement-vs-truth separation — `.claude/skills/mercado-livre-listing-best-practices/references/compliance.md`
  — internal — architectural reference only.
- Full detail: `research/shopee-listing-skill/discovery-report.md` §17, §21,
  §22, §29 (U9, U12).
