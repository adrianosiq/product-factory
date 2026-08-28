# research/shopee-listing-skill — DISCOVERY ARTIFACT

**Status: Phase 01 — Discovery & Architecture. NOT a Skill.**

This directory holds the research output that must exist *before*
`.claude/skills/shopee-listing-best-practices/` is written. Nothing here is a
rule the Product Factory agents should follow yet — it is a map of what Shopee
Brasil's listing model appears to be, how well each fact is verified, and which
Product Factory concepts look shared vs marketplace-specific.

| File | What it is |
|---|---|
| `discovery-report.md` | The full 31-section discovery report, including the fact table, API table, entity table, dynamic-check table, Mercado Livre comparison, proposed Skill structure, unresolved gaps, and the Phase 01 decision. |

**Verification reality (read first):** the Shopee Open Platform API portal
(`open.shopee.com`) is unreachable from the research environment, and the Shopee
Brasil Seller Education Hub / Central de Ajuda render client-side and expose only
page titles to a fetch. Every "OFFICIAL" Shopee fact in the report is therefore
`SEARCH_INDEXED` (reconstructed from search snippets + third-party integrator
docs + community SDKs), and all API field/limit detail is `UNVERIFIED`. This
mirrors the Mercado Livre Skill's `⚠ verify` situation and must be resolved in
Phase 02 before any rule is locked.

Decision: **DISCOVERY GAPS MUST BE RESOLVED FIRST** — see report §31 / §65.
