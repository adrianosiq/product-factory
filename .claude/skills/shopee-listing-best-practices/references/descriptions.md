# Descriptions

last_reviewed: 2026-08-28
phase_02_3_reviewed: 2026-09-03
volatile: true
classification: OFFICIAL (policy) — length-limit source PRIMARY_VERIFIED (SPD-029, DYNAMIC values); content policy SEARCH_INDEXED
phase_02_3_note: >-
  PRIMARY (SPD-029, SPD-010). **`description` length limit is
  `get_item_limit.item_description_length_limit {min,max}`** (DYNAMIC; doc
  **sample** min 10 / max 2000 — a sample, not a constant). The prior ≈5,000
  fixed guess has **no primary basis and is superseded** by this dynamic source —
  never assume a fixed value. `extended_description` is a
  separate object bounded by `get_item_limit.extended_description_limit` and is
  **whitelist sellers only** (`description_type = extended`, else `normal`).
  Hashtags in the description ≤ 18 (`error_desc_hash_tag_over_limit`).
  `error_desc_length_min_limit` on violation.

## 1. What is known (length source = Phase 02.3 primary; content policy = `SEARCH_INDEXED`)

| Rule | Resolution | Status |
|---|---|---|
| `description` length limit | **`get_item_limit.item_description_length_limit {min, max}`** — resolve at listing time | **`PRIMARY_VERIFIED` source** (SPD-029). The prior ≈5,000 fixed guess has **no primary basis** and is superseded; doc **sample** min 10 / max 2000 (a sample, not a constant). |
| Format | `normal` plain text baseline; `description_type = extended` gives an image+text block layout, bounded by `get_item_limit.extended_description_limit` — **whitelist sellers only** | OFFICIAL (SPD-029); BR whitelist status `UNVERIFIED` |
| Prohibited content | external links, phone numbers, QR codes, social handles, "chame no WhatsApp" / any off-Shopee diversion; seller contact info; misleading / inaccurate claims | OFFICIAL, `SEARCH_INDEXED` |
| Recommended content | dimensions, material, usage, what's in the box, warranty / return terms; structured and scannable | OFFICIAL (recommendation), `SEARCH_INDEXED` |
| `normal` vs `extended` bodies | differ; which BR categories support `extended` is `⚠ verify` | — | `UNVERIFIED` |

## 2. How the Skill treats it

- **Length limit unresolved** → `PUBLICATION_STATUS = REVIEW`
  (`resolve_description_limit`). Do not assume any fixed cap; only a limit
  resolved from `get_item_limit` that the description exceeds →
  `PUBLICATION_STATUS = FAIL` (`error_desc_length_min_limit` / over max).
- **Prohibited contact / external-diversion strings** are **removable
  content-policy violations**: drop the string → `CONTENT_STATUS` may stay
  `PASS`; `PUBLICATION_STATUS = FAIL` only if the assembled payload still carries
  it at publish (`references/compliance.md`).
- **Claim safety is independent of formatting.** Every material claim in the
  description traces to `CONFIRMED` evidence (SKILL.md §5); no invented feature,
  spec, certification or benefit. An unsupported *removable* claim is dropped
  (`CONTENT` may stay `PASS`); an unsupported *essential* claim → `REVIEW` / `FAIL`
  by evidence.
- **Do not rely on `extended`** until BR + category support is confirmed; draft
  for `normal` plain text and note `extended` as an optional enhancement.
- Description quality (structure, completeness, scannability) → `QUALITY_STATUS`,
  not `PUBLICATION_STATUS`.

## 3. Consistency

The description must not contradict the title, attributes, images or variant
table (material, model, quantity, colour, inclusions). Any material mismatch →
`BLOCKER — PRODUCT DATA CONFLICT` (`affects: [CONTENT, PUBLICATION]`).

## Sources

- Description ≈5,000 chars; no links / phones / QR / social / WhatsApp;
  `normal` / `extended` types — BR integrators (`base.com`, `anymarket`,
  `ideris`, `maino`), `help.shopee.com.br` — external / Central de Ajuda —
  consulted 2026-08-28 — `SEARCH_INDEXED`, MEDIUM.
- Recommended structured content — `seller.shopee.com.br/edu` — Centro de
  Educação — consulted 2026-08-28 — `SEARCH_INDEXED`.
- Removable-string handling — `.claude/skills/mercado-livre-listing-best-practices/references/compliance.md`
  — internal — architectural reference only.
- Full detail: `research/shopee-listing-skill/discovery-report.md` §11, §29 (U5).
