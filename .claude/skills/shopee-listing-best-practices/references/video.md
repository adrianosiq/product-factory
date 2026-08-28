# Video — surface-scoped

last_reviewed: 2026-08-28
volatile: true
classification: OFFICIAL (existence) — verification SEARCH_INDEXED; listing-video constraints `UNVERIFIED`

## 1. Why this file is separate

Three different Shopee surfaces carry video, each with **its own policy**:

| Surface | `scope` | Rules found | Verification |
|---|---|---|---|
| **Product listing video** | `listing` | one video on the listing (`media_space/upload_video` → `video_upload_id`). Duration / size / aspect for the *listing* video: **`UNVERIFIED` for BR.** | `SEARCH_INDEXED` (existence) |
| **Shopee Video** (short-form social feed) | `shopee_video` | ≈60 s max (≤30 s preferred), **9:16 vertical**, no borders, one product in focus, creator originality tiers, community guidelines | `SEARCH_INDEXED` |
| **Shopee Live** | `shopee_live` | its own prohibited-products policy (help art. 188686) and its own content rules | `SEARCH_INDEXED` |

## 2. Rule

**Every video rule carries a `scope` field.** A Shopee Video or Shopee Live rule
must **not** be applied to the product listing unless Shopee explicitly makes it
cross-surface. The same caution applies to prohibited-content, image, title,
link and regulated-goods rules discovered on the Live / Video surfaces.

- Do **not** transplant the 60 s / 9:16 Shopee Video numbers onto the listing
  video.
- The listing video's real constraints are an open question (SKILL.md gap G8);
  until resolved, mark listing-video specs unknown and treat the video as an
  optional enhancement.

## 3. Product Identity Guard applies

A listing video is a presentation asset. It must not introduce a product fact the
evidence does not support (a feature, an included accessory, a variant that is
not the actual variant). Same `IDENTITY_PASS` / `REVIEW` / `FAIL` audit as
images (`references/images.md` §D).

## 4. Readiness impact

- Missing listing video → `QUALITY_STATUS = REVIEW` at most; never a publication
  blocker.
- Listing-video spec constraints unresolved → note as pending; do not `FAIL`.
- A video that misrepresents the product → identity `BLOCKER` /
  `REVIEW` as in `images.md`.

## Sources

- Listing video via `media_space/upload_video` → `video_upload_id` —
  `github.com/wjp-letgo/shopeego` — community SDK — consulted 2026-08-28 —
  `SEARCH_INDEXED`; constraints `UNVERIFIED`.
- Shopee Video 60 s / 9:16 / guidelines — `seller.shopee.com.br/edu` (art.
  20631), `ads.shopee.com.br/learn` — Centro de Educação / Shopee Ads —
  consulted 2026-08-28 — `SEARCH_INDEXED`.
- Shopee Live prohibited-products policy — `help.shopee.com.br` art. 188686 —
  Central de Ajuda — consulted 2026-08-28 — `SEARCH_INDEXED`.
- Full detail: `research/shopee-listing-skill/discovery-report.md` §24, §29 (U14).
