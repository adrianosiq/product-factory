# Moderation & enforcement — post-publication, separate from ProductMaster truth

last_reviewed: 2026-08-28
phase_02_2_reviewed: 2026-08-30
volatile: true
classification: OFFICIAL (existence) — verification SEARCH_INDEXED; mechanics & API `⚠ verify` / `UNVERIFIED`
phase_02_2_note: >-
  Per-item violation and content-diagnosis method names now exist
  (`get_item_violation_info`, `get_item_content_diagnosis_result`,
  `get_item_list_by_content_diagnosis`) — `SEARCH_INDEXED`; BR availability and
  schema `UNVERIFIED`. `item_status` enum values NOT re-verified. See
  `research/shopee-api-contract/phase-02.2-report.md` §23–§24.

## 1. Why this file is separate

Phase 01 found seller / listing enforcement to be prominent on Shopee BR
(penalty points, bans). It must stay **separate from pre-publication readiness**
and **separate from `ProductMaster` truth**. Enforcement state is an `EXECUTION`
/ compliance input, never a `CONTENT` input.

## 2. Listing status graph

`item_status` (v2) — corroborated set:

| State | Meaning (reconstructed) | Verification |
|---|---|---|
| `NORMAL` | live and for sale | `SEARCH_INDEXED` |
| `UNLIST` | exists but hidden / not for sale (seller toggled) | `SEARCH_INDEXED` |
| `BANNED` | removed by Shopee for a policy violation | `SEARCH_INDEXED` |
| `DELETED` | removed (by seller or system) | `SEARCH_INDEXED` |
| `REVIEWING` (?) | pending moderation after create / edit — possibly a separate `review_status` | `UNVERIFIED` |

Provisional graph: `NOT_CREATED → (create) → [REVIEWING?] → NORMAL ⇄ UNLIST`,
with `BANNED` / `DELETED` terminal-ish. Whether edits re-trigger review —
`⚠ verify`. Full transition graph — `⚠ verify` (SKILL.md gap, lifecycle).

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

- `item_status = BANNED` is visible via `get_item_base_info` / `get_item_list`.
- **Phase 02.2 correction:** a **per-item violation** method name now exists in
  community SDKs — `product/get_item_violation_info` (`item_id`) — and a
  **content-diagnosis** pair — `product/get_item_content_diagnosis_result`
  (`item_id`) and `product/get_item_list_by_content_diagnosis` (`diagnosis_status`).
  Phase 01 said "no per-item violation API"; that is corrected to
  `SEARCH_INDEXED` (schema + BR availability still `UNVERIFIED`).
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
- `item_status = BANNED` / `DELETED` for an update operation →
  `EXECUTION_STATUS = FAIL` for that operation.
- Never recreate a listing to dodge a ban. Preserve `reason` + `remedy`; never
  fabricate a remedy.
- "No moderation found" is **not** proof a payload is compliant.

## Sources

- `item_status` enum — `github.com/wjp-letgo/shopeego` — community SDK —
  consulted 2026-08-28 — `SEARCH_INDEXED`.
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
