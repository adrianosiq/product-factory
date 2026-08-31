# research/shopee-primary-docs — Phase 02.3 Primary Documentation Ingestion

**Status (2026-08-30): `PRIMARY DOCUMENTATION REQUIRED` — no primary artifact has
been supplied or is reachable. Nothing has been ingested.**

## What this phase is

Phase 02.2 produced a **credible, corroborated map** of the Shopee Open Platform
v2 `product` API — but every implementation-critical fact is still
`SEARCH_INDEXED` / `UNVERIFIED` because no primary Shopee source could be read
(`open.shopee.com`, `developer.shopee.com` and `web.archive.org` are blocked at
the fetch-tool level; the portal is a client-rendered SPA).

Phase 02.3 does **not** repeat broad web research. It exists to
**obtain → preserve → classify → extract → map → reconcile PRIMARY Shopee
documentation** so that individual claims can move from
`SEARCH_INDEXED` / `UNVERIFIED` to `LIVE` / `PRIMARY_VERIFIED` where the primary
text directly supports them.

Community SDKs, integrator docs, forums, Stack Overflow, third-party API
catalogues and search snippets **must not** be used to upgrade an
implementation-critical rule in this phase. They may be consulted **only** to
locate the corresponding primary document.

## Files

| File | Purpose |
|---|---|
| `evidence-registry.md` | One row per supplied primary artifact (`SPD-xxx`): provenance, market, version, completeness, primary status. Currently **empty**. |
| `claim-registry.md` | One row per implementation-critical claim (`SCL-xxx`): previous state, evidence, new state, scope. Seeded from Phase 02.2; all claims currently **awaiting primary evidence**. |
| `extraction-report.md` | Per-contract-area findings from ingested artifacts + the **P0 completion matrix** + the **acquisition checklist**. Currently a checkpoint: nothing ingested. |
| `reconciliation-report.md` | Phase 02.2 claim ⇄ primary evidence, each classified `PRIMARY_CONFIRMED` / `CORRECTED` / `PARTIALLY_CONFIRMED` / `STILL_UNVERIFIED` / `CONFLICTING` / `OUT_OF_SCOPE`. Currently all `STILL_UNVERIFIED`. |
| `artifacts/` | Raw supplied artifacts, **only if** repository/security policy permits and after a secret scan + redaction. Otherwise keep artifacts out-of-repo and record only their `evidence-registry.md` metadata. |

## How a maintainer resumes this phase

1. Acquire the P0 documents in `extraction-report.md` §"Acquisition checklist"
   (PDF / HTML export / print-to-PDF / screenshots / API Explorer captures).
2. **Run a secret scan** (`partner_key`, `access_token`, `refresh_token`, auth
   `code`, cookies, session ids, client secrets, private keys, seller PII) and
   **redact** before anything enters the repo. Never use real credentials as
   examples.
3. Add one `SPD-xxx` row per artifact to `evidence-registry.md` with full
   provenance. Establish `Primary status` (`VERIFIED_PRIMARY` /
   `CANDIDATE_PRIMARY` / `REJECTED`) from **contents**, not the file name.
4. Extract **field-level** facts into `extraction-report.md`; upgrade individual
   claims in `claim-registry.md` (never bulk-upgrade an endpoint because its page
   arrived).
5. Fill `reconciliation-report.md`; update the P0 completion matrix.
6. Update the Shopee Skill **only** for claims that became `PRIMARY_VERIFIED` or
   were directly corrected — with a traceable `evidence ID` + scope + date.
7. Re-run the Phase 02.3 final-decision test (`extraction-report.md` §"Final
   decision").

## Scope guard

Target = **the minimum primary evidence needed to safely lock the Product
Factory listing contract.** Do not collect Orders / Returns / Ads / Affiliate /
Chat / Finance documentation unless a listing-contract dependency requires it.
