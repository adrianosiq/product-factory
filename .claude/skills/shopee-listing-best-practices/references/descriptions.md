# Descriptions

last_reviewed: 2026-08-28
volatile: true
classification: OFFICIAL (policy) — verification SEARCH_INDEXED; length limit DYNAMIC and NOT locked

## 1. What discovery found (all `SEARCH_INDEXED`, `⚠ verify`)

| Rule | Discovery value | Status |
|---|---|---|
| Long-description limit | ≈ **5,000** characters (BR) | **DYNAMIC — resolve via `get_item_limit` / live docs. NOT an enforced constant.** `SEARCH_INDEXED`, MEDIUM. |
| Format | plain text baseline; `description_type = extended` gives an image+text block layout in some flows | OFFICIAL; availability for BR `UNVERIFIED` |
| Prohibited content | external links, phone numbers, QR codes, social handles, "chame no WhatsApp" / any off-Shopee diversion; seller contact info; misleading / inaccurate claims | OFFICIAL, `SEARCH_INDEXED` |
| Recommended content | dimensions, material, usage, what's in the box, warranty / return terms; structured and scannable | OFFICIAL (recommendation), `SEARCH_INDEXED` |
| `normal` vs `extended` bodies | differ; which BR categories support `extended` is `⚠ verify` | — | `UNVERIFIED` |

## 2. How the Skill treats it

- **Length limit unresolved** → `PUBLICATION_STATUS = REVIEW`
  (`resolve_description_limit`). Do not enforce ≈5,000 as a hard cap; only a
  resolved limit the description exceeds → `PUBLICATION_STATUS = FAIL`.
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
