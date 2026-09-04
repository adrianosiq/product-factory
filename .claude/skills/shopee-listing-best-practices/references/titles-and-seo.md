# Titles & SEO

last_reviewed: 2026-08-28
phase_02_3_reviewed: 2026-09-03
volatile: true
classification: mixed — see per-rule tags; title length is DYNAMIC and NOT locked
phase_02_3_note: >-
  PRIMARY (SPD-029 get_item_limit, SPD-010 add_item). Title = `item_name`.
  **Length limit is `get_item_limit.item_name_length_limit {min_limit,
  max_limit}`** — per shop+category, DYNAMIC. The ≈255/256 value is REJECTED
  (doc sample is min 5 / max 100). Primary errors: `error_title_exceeds_max_length`,
  `error_item_name_is_too_short`, `error_title_character_forbidden`,
  `error_name_length_limit`, `error_item_name_empty`. SEO/ranking claims stay
  EXPERIMENTAL (not in the corpus); `get_item_base_info.deboost` is a real
  "search ranking lowered" flag. See `research/shopee-primary-docs/`.

## 1. Title — four separate layers (keep them apart)

### A. OFFICIAL hard constraints (verification `SEARCH_INDEXED`, `⚠ verify`)

| Rule | Discovery value | Status |
|---|---|---|
| Max `item_name` length | ≈ **255–256** characters (BR) | **DYNAMIC — resolve via `get_item_limit`. NOT an enforced constant.** `SEARCH_INDEXED`, MEDIUM. Do not use the ~60/120 of older SEA markets. |
| Min length | ≈ 10–25 chars (varies by source) | `UNVERIFIED` — `DYNAMIC` |
| Prohibited characters | no special characters; no leading / trailing spaces | OFFICIAL, `SEARCH_INDEXED` |

Enforcement note: until `resolve_title_limit` is executed, an over-length title
is `PUBLICATION_STATUS = REVIEW` (check pending), **not** `FAIL`. Only a resolved
limit that the title exceeds → `FAIL`.

### B. OFFICIAL content policy (verification `SEARCH_INDEXED`)

- No seller contact info or social handles; no external diversion.
- No misleading / inaccurate keywords; no keyword spam; no unrelated brand names.
- No promotional terms in the title ("Frete Grátis", "Promoção", "Desconto 50%",
  …).
- Case: **Title Case** (capitalise each word); avoid ALL CAPS. OFFICIAL
  *recommendation*.

A prohibited string is a **removable content-policy issue**: drop it →
`CONTENT_STATUS` may stay `PASS`; `PUBLICATION_STATUS = FAIL` only if the
assembled payload still carries it at publish (`references/compliance.md`).

### C. OFFICIAL recommendations (shape)

Concise + informative: include brand, product line, model, and distinguishing
specs (material, main ingredient, colour, size, quantity) **when they legitimately
exist and are evidence-backed**. Never invent a spec to fill a template.

### D. INTERNAL strategy / EXPERIMENTAL SEO

- "Produto + Marca + Características + Modelo" ordering — INTERNAL convention
  (from third parties), **not** a Shopee rule.
- "First ≈65 characters are what shows before the fold" — `EXPERIMENTAL` /
  `LEARNED`; third-party, **not** an official rule and **not** a stated ranking
  factor.

## 2. SEO — discipline

Shopee officially states (S2 / S4, `SEARCH_INDEXED`) that listing
**completeness and quality** — title, images, description, correct category,
product code, attributes — plus relevance, conversion/sales, ratings, seller
performance / penalty points, price competitiveness, shipping options and Ads
**influence exposure**. "Well-filled listings get more exposure" is stated
directly → this may be documented as an **OFFICIAL recommendation**.

**Ranking interpretation stays `EXPERIMENTAL`** unless Shopee states the
relationship explicitly. Do **not** assert:

- "the first N characters rank better",
- "keyword density improves ranking",
- "description keywords guarantee exposure",
- "3:4 images rank higher", "attribute-count thresholds lift impressions".

"Shopee recommends X" must **not** silently become "X is a ranking factor".

| Signal group | Examples | Tag |
|---|---|---|
| stated to affect exposure | listing completeness, category correctness, attribute fill, image compliance, relevance, sales/conversion, ratings, penalty points, shipping | OFFICIAL (`SEARCH_INDEXED`) |
| from our data (future) | which completeness elements move impressions for our catalogue | LEARNED |
| hypothesis | title keyword ordering, first-N-chars weighting, image-ratio ranking lift | EXPERIMENTAL |

## 3. Readiness impact

- Title-length limit unresolved → `PUBLICATION_STATUS = REVIEW`
  (`resolve_title_limit`).
- Resolved limit exceeded → `PUBLICATION_STATUS = FAIL`.
- Prohibited character / promo term / contact string in the title → drop it;
  `MAJOR` (or `BLOCKER` on `PUBLICATION` if still present at publish).
- Invented brand / model / spec in the title → `BLOCKER` (`CONTENT` +
  `PUBLICATION`).
- Weak term order / thin title / missing legitimate spec → `QUALITY_STATUS =
  REVIEW`, never a publication blocker.

## Sources

- Title length ≈255–256, Title Case, no special chars, no promo terms —
  `seller.shopee.com.br/edu`, BR integrators (`base.com`, `anymarket`, `ideris`,
  `maino`) — Centro de Educação / external — consulted 2026-08-28 —
  `SEARCH_INDEXED`, MEDIUM (length LOW–MEDIUM).
- Completeness / quality influences exposure — `seller.shopee.com.br/edu`,
  `ads.shopee.com.br/learn` — Centro de Educação / Shopee Ads — consulted
  2026-08-28 — `SEARCH_INDEXED`.
- "First ~65 chars before the fold" — BR educators — external — consulted
  2026-08-28 — `SEARCH_INDEXED`; EXPERIMENTAL.
- SEO discipline — `.claude/skills/mercado-livre-listing-best-practices/references/seo-and-discovery.md`
  — internal — architectural reference only.
- Full detail: `research/shopee-listing-skill/discovery-report.md` §9, §10.
