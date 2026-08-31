# Primary Evidence Registry — Phase 02.3

last_updated: 2026-08-30
status: **EMPTY — no primary artifact supplied.**

One row per supplied primary artifact. Assign a stable `Evidence ID`
(`SPD-001`, `SPD-002`, …) in order of receipt; never renumber.

## Registry

| Evidence ID | Title | Source surface | Original locator | Market | API version | Captured at | Authentication | Artifact type | Completeness | Primary status | Supports | Notes |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| — | *(none received)* | — | — | — | — | — | — | — | — | — | — | Phase 02.3 is blocked on artifact supply. |

## Column meanings

| Field | Meaning |
|---|---|
| **Evidence ID** | Stable internal identifier (`SPD-xxx`). |
| **Title** | The official document / page title, verbatim. |
| **Source surface** | `Open Platform docs` / `Developer console` / `API reference` / `API Explorer` / `Seller Education` / `Onboarding/Application` / `Official SDK-sample` / `Official response example`. |
| **Original locator** | URL or `open.shopee.com/documents/v2/...` doc-locator when known. |
| **Market** | `GLOBAL` / `BRAZIL` / `OTHER` / `UNKNOWN_MARKET_SCOPE`. |
| **API version** | e.g. `v2`, `v1`, `n/a`. |
| **Captured at** | Date the maintainer exported / screenshotted it. |
| **Authentication** | `PUBLIC` / `AUTHENTICATED` / `UNKNOWN` — was login required to view it. |
| **Artifact type** | `PDF` / `HTML export` / `Markdown` / `plain text` / `screenshot` / `print-to-PDF` / `API Explorer capture`. |
| **Completeness** | `FULL` / `PARTIAL` / `SCREENSHOT`. Screenshots and partial exports only prove the fields visible in them. |
| **Primary status** | `VERIFIED_PRIMARY` (contents clearly identify an official Shopee developer source) / `CANDIDATE_PRIMARY` (looks official, provenance not yet confirmed) / `REJECTED` (reproduction of endpoint names without official provenance). |
| **Supports** | The specific claims / resources / fields this artifact covers. |
| **Notes** | Limitations, redactions applied, version/market caveats. |

## Rules

- **File names prove nothing.** `shopee-official-api.pdf` is not primary until its
  contents establish official Shopee provenance and the maintainer records where
  it came from.
- A copied API page **can** be primary if the content clearly identifies the
  official Shopee developer source and the origin is recorded.
- **Never** commit `partner_key`, `access_token`, `refresh_token`, authorization
  `code`, cookies, session ids, client secrets, private keys, or seller PII.
  Redact before storage; if artifacts cannot be safely committed, keep them
  out-of-repo and record only this metadata.
- Screenshots → `completeness = SCREENSHOT`; do not infer fields outside the
  frame, and do not reconstruct missing portions from community SDKs.
