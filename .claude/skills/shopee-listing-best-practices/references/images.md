# Images

last_reviewed: 2026-08-28
phase_02_3_reviewed: 2026-09-03
volatile: true
classification: mixed — Shopee specs OFFICIAL (verification SEARCH_INDEXED, `⚠ verify`); numeric values NOT locked
phase_02_3_note: >-
  PRIMARY (SPD-029, SPD-010). Image **count** = `get_item_limit.item_image_count_limit
  {min,max}` (DYNAMIC; doc sample 1–9). `add_item.image_ratio`: `"1:1"` (default)
  or `"3:4"` — **3:4 is whitelisted sellers only**. `promotion_images`: one
  image, ratio **must be 3:4**. Upload via `v2.media_space.upload_image`
  (page NOT in the corpus) → persist `image_id`. Pixel min/max, byte cap, file
  types, cover-photo moderation rules are `PRIMARY_NOT_FOUND` (media_space /
  seller-education pages not supplied) — keep A/B layers `⚠ verify`. Layer D
  Product Identity Guard is INTERNAL, unchanged.

Keep four layers separate. Do not let a creative preset be read as a Shopee rule,
and do not let a provisional number be read as a locked constant.

## A. Shopee official requirements (verification `SEARCH_INDEXED`, `⚠ verify`)

| Rule | Discovery value | Confidence / status |
|---|---|---|
| Image count | **1–9** images | `SEARCH_INDEXED`, HIGH (consistent) — still `⚠ verify`; treat as `DYNAMIC` via `get_item_limit` |
| Aspect ratio | **1:1 mandatory** for product images | `SEARCH_INDEXED`, MEDIUM–HIGH |
| Min dimensions | stated variously ≈ **350×350** to 500×500 / 800×800 | `SEARCH_INDEXED`, LOW–MEDIUM — **do not hardcode**; `DYNAMIC` via `get_item_limit` |
| File types | JPG / PNG (likely) | exact set `UNVERIFIED` |
| Byte cap | ≈ 2 MB/image (SEA docs) | `OTHER_MARKET_REFERENCE`, `UNVERIFIED` for BR |
| Cover / main image | no commercial text ("Frete Grátis", "50% Off", …); no watermark; no border; no logo/mascot; no Shopee-brand elements; no sensitive content; white background (recommended→required, varies by source) | `SEARCH_INDEXED`, MEDIUM–HIGH |
| Product-as-logo rule | if a product photo is used as the shop/brand logo, the product must fill **≥ 60%** of the frame, centered | `SEARCH_INDEXED`, MEDIUM–HIGH |
| Image per variation | tier-1 options may each carry one image | `SEARCH_INDEXED` |
| Moderation | automated + manual; 2026 rules tightened — non-compliant images → listing banned / hidden | `SEARCH_INDEXED` |

**None of 1–9 / 1:1 / 3:4 / ≈350×350 / ≥60% / cover-text-ban is a final locked
rule.** Each is recorded with its discovery evidence, classification and
verification state, and resolved at listing time (`resolve_image_limits`).

## B. Shopee official recommendations

- **3:4** ratio — optional, recommended for extra search exposure. Recommendation
  only; missing → `QUALITY_STATUS = REVIEW`, never a publication blocker.
- Recommended dimensions ≈ **1024×1024** (some sources ≈ 1200×1200) — INTERNAL /
  recommendation, `SEARCH_INDEXED`, LOW confidence.
- Clean, well-lit, product-filling main image; additional angles, scale,
  in-use, detail, and an "in the box" shot.

## C. INTERNAL commercial gallery strategy

- A plan of *useful* images (main, angles, scale/dimension, detail, in-use,
  in-the-box, variant shots) — a guideline, **not** a quota. No filler to hit a
  number.
- Creative presets (fill %, tilt angles, backgrounds) are **INTERNAL** and never
  presented as Shopee policy.

## D. Product Identity Guard (`SHARED_CORE_CANDIDATE`, INTERNAL — never an OFFICIAL Shopee rule)

> **Presentation may change. Product identity may not.**

Protect, where applicable: geometry, colour, material appearance, finish,
components, quantity, branding, included accessories, packaging contents, variant
identity, scale.

- Run an identity audit on **every** generated / edited asset:
  `IDENTITY_PASS` / `IDENTITY_REVIEW` / `IDENTITY_FAIL`.
- A generated / edited image is an **output, never evidence** for a newly
  introduced product fact.
- `IDENTITY_FAIL` (geometry / colour / material / component / condition altered;
  a fabricated feature or accessory shown as included; a generated variant that
  is not the actual variant; an image contradicting `ProductMaster`) →
  `BLOCKER` on `PUBLICATION` (and `CONTENT` where truthful representation becomes
  impossible).
- `IDENTITY_REVIEW` (insufficient source evidence for a generated detail;
  uncertain colour / scale; a reconstructed angle exposing unseen areas; a
  lifestyle composition implying an unsupported inclusion) → `REVIEW`.

Shopee's tightened 2026 image moderation and its return reasons (item not as
described, wrong variation, missing parts, size / material / colour mismatch)
both punish exactly the gap the Guard protects.

## Readiness impact

- Image limits unresolved → `PUBLICATION_STATUS = REVIEW` (`resolve_image_limits`).
- 0 compliant 1:1 images, or an image over/under a **confirmed hard** bound, or a
  cover-photo moderation rule broken → `PUBLICATION_STATUS = FAIL`.
- Recommended-only spec missed (3:4, ≈1024, RGB) → `QUALITY_STATUS = REVIEW`.
- Identity audit results as above.

## Sources

- Count 1–9, 1:1 mandatory, 3:4 recommended, cover-photo text ban, ≥60% frame
  rule, 2026 moderation tightening — `seller.shopee.com.br/edu` (arts. 17369,
  3304), BR integrators (`base.com`, `gobots`, `mambadigital`) — Centro de
  Educação / external — consulted 2026-08-28 — `SEARCH_INDEXED` (count HIGH;
  dimensions LOW).
- Min ≈350×350 / rec ≈1024×1024, ≈2 MB cap — BR integrators, SEA docs —
  external — consulted 2026-08-28 — `SEARCH_INDEXED` / `OTHER_MARKET_REFERENCE`,
  LOW confidence.
- Three-layer model + Product Identity Guard — `.claude/skills/mercado-livre-listing-best-practices/references/images.md`
  — internal — architectural reference only.
- Full detail: `research/shopee-listing-skill/discovery-report.md` §12, §23,
  §29 (U4).
