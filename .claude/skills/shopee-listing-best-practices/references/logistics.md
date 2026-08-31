# Logistics — placeholders to resolve

last_reviewed: 2026-08-28
phase_02_2_reviewed: 2026-08-30
volatile: true
classification: OFFICIAL (fields exist) — verification SEARCH_INDEXED; requirements & limits `UNVERIFIED`
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
