# Logistics — placeholders to resolve

last_reviewed: 2026-08-28
phase_02_3_reviewed: 2026-09-03
phase_02_2_reviewed: 2026-08-30
volatile: true
classification: OFFICIAL (fields exist) — verification SEARCH_INDEXED; requirements & limits `UNVERIFIED`
phase_02_3_note: >-
  PRIMARY (SPD-029 get_item_limit, SPD-010 add_item). **DTS + weight + dimension
  + size-chart requiredness ARE in `get_item_limit`** — CORRECTS Phase 02.2's
  "logistics service, not get_item_limit": `days_to_ship_limit {min, max,
  non_pre_order_days_to_ship}` (category), `weight_limit {weight_mandatory}`,
  `dimension_limit {dimension_mandatory}`, `size_chart_limit {size_chart_mandatory,
  support_image_size_chart, support_template_size_chart}`. `add_item`:
  `weight` (KG, required), `dimension {package_length/width/height}` cm,
  `logistic_info[{logistic_id (req), enabled (req), is_free, size_id (when
  fee_type = SIZE_SELECTION), shipping_fee (when fee_type = CUSTOM_PRICE)}]`,
  `pre_order {is_pre_order, days_to_ship}`, `condition` NEW/USED. **Still
  MISSING (`PRIMARY_NOT_FOUND` — pages not supplied):** `v2.logistics.get_channel_list`,
  `v2.logistics.get_address(_list)`, `v2.shop.get_warehouse_detail`, a dedicated
  `get_dts_limit` page. `error_category_dts` / `error_param_dts_exceeds_max_limit`
  / `error_param_category_not_support_pre_order` on violation.
phase_02_2_note: >-
  Days-to-ship / handling-time limits live in a `logistics` service, not the
  `product` service — Phase 02.1's `get_dts_limit` under `product` is corrected.
  `logistics/get_channel_list` and `logistics/get_address` corroborated. See
  `research/shopee-api-contract/phase-02.2-report.md` §21, §29 (C3).

## 1. What is (weakly) known

| Aspect | Finding | Status |
|---|---|---|
| Weight | `weight` (kg) at item level; appears **required to publish** | `SEARCH_INDEXED`, MEDIUM |
| Package dimensions | `{ package_length, package_width, package_height }` (cm); may be required per channel / category | `SEARCH_INDEXED` |
| Channels | `logistics/get_channel_list` → channels enabled for the shop; at least one must be enabled on the listing | `SEARCH_INDEXED` |
| Addresses | `logistics/get_address` → pickup / return / default `address_id` (a shop setting) | `SEARCH_INDEXED` |
| Handling time / days-to-ship | `days_to_ship`; category limits via a **`logistics`-service** resource — **not** a `product` resource (Phase 02.2 corrects Phase 02.1's `get_dts_limit` filing; exact name `UNVERIFIED`) | `SEARCH_INDEXED` / `UNVERIFIED` |
| Weight recommendation | `product/get_weight_rec` (a recommendation, not a limit) | `SEARCH_INDEXED` (S12) |
| Pre-order | `pre_order { is_pre_order, days_to_ship }`; a longer ship window (v1 said 7–30) | `SEARCH_INDEXED`; numbers `⚠ verify` |
| Condition | `condition ∈ NEW | USED`; USED appears category-gated in BR | `SEARCH_INDEXED`, LOW–MEDIUM |
| Fulfilment | Shopee BR operates a growing DC network / "Envios Shopee"; DC stock not seller-writable | `SEARCH_INDEXED` (press) |

## 2. Open questions (do not universalise Phase 01 assumptions)

Resolve, per shop / category, before enforcing:

- **weight / dimensions** — exactly when each is mandatory (channel- vs
  category-driven),
- **channels** — which are available for a BR shop; free-shipping program
  interaction,
- **pickup vs drop-off** — address requirements,
- **fulfilment** — Envios Shopee eligibility and its effect on stock / handling,
- **pre-order** — allowed window, category restrictions, exact `days_to_ship`
  bounds,
- **dangerous goods** — is there an `item_dangerous` flag; which categories;
  what documentation,
- **handling time** — category `get_dts_limit` min / max.

## 3. Readiness impact

- No enabled logistics channel, or missing weight, in a resolved context →
  `PUBLICATION_STATUS = FAIL` (`resolve_logistics`).
- Logistics context unresolved → `PUBLICATION_STATUS = REVIEW`.
- `days_to_ship` outside a resolved `get_dts_limit` range → `PUBLICATION_STATUS =
  FAIL` (`resolve_dts_limit`).
- `condition = USED` where the resolved category disallows it →
  `PUBLICATION_STATUS = FAIL` (`resolve_condition_allowed`); also a compliance
  check (`references/compliance.md`).
- Dangerous-goods / regulated-transport applicability → compliance finding.

## Sources

- `weight`, `dimension`, `logistic_info`, `days_to_ship`, `pre_order` 7–30,
  `condition` — `github.com/teacat/shopeego` (v1), `github.com/wjp-letgo/shopeego`
  (v2) — community SDKs — consulted 2026-08-28 — `SEARCH_INDEXED` (v1 numbers
  stale).
- `logistics/get_channel_list`, `logistics/get_address` — `developer.inlinex.com.sg`
  — external — consulted 2026-08-28 — `SEARCH_INDEXED`.
- BR DC network / Envios Shopee — BR press — external — consulted 2026-08-28 —
  `SEARCH_INDEXED`.
- USED category-gating — `help.shopee.com.br` — Central de Ajuda — consulted
  2026-08-28 — `SEARCH_INDEXED`, LOW–MEDIUM.
- Full detail: `research/shopee-listing-skill/discovery-report.md` §14, §15,
  §"Fact Table", §29 (U15).
