# SEO & discovery

last_reviewed: 2026-08-27

Mercado Livre is a marketplace, not a generic web search engine. Describe the
**real mechanism** — category routing, structured-attribute filters, product
matching, listing quality / relevance — not web-SEO folklore (meta keywords,
keyword density, backlinks, hidden text). This file keeps the name "SEO" for
familiarity only. ML does not publish a full ranking-factor list; keep what ML
actually says separate from our strategy and from untested hypotheses.

## OFFICIAL (⚠ verify)

- **Category** is the primary routing signal — a listing appears under its
  category tree and that tree's filters.
- **Structured attributes / ficha técnica are what ML uses to represent product
  characteristics and to power category filters and product matching.**
- **Incomplete listings lose relevance** and may be sent to internal review until
  the missing fields are filled (Central de Vendedores — "O status das suas
  fichas técnicas").
- ML frames a **complete, high-quality listing** — clear title, good photos,
  competitive commercial conditions — as better for exposure, without publishing
  the ranking factors behind it.
- A valid **universal identifier** helps match a listing to the catalog.

## INTERNAL STRATEGY

- Fill every relevant, evidence-backed structured attribute. An attribute you
  leave blank is a filtered shopping journey you most likely won't appear in —
  incomplete product data reduces the listing's ability to participate in
  filtered / faceted browsing. (This is our framing of *why* to be thorough; what
  ML documents is that incomplete fichas lose relevance, not an absolute
  "invisible" rule.)
- Lead the title / `family_name` with the most product-defining terms for
  **clarity and scannability** (see `titles-and-family-name.md`) — treated as a
  readability choice, not a known ranking lever.
- Prefer a real structured attribute over stating the same fact only in
  free-text description (structured is filterable; description is not).
- Use the buyer's real vocabulary (from `review-mining.md` /
  `competitor-research.md`) in the description and open attributes, without
  stuffing — this is terminology alignment for comprehension and query match,
  not an algorithm hack.
- Keep brand / model spelling identical across title, attributes and description
  so one query matches all of them.

## EXPERIMENTAL (must be tested with performance data)

- Whether front-loading the most defining terms in the title measurably affects
  discovery or CTR (the testable form of the retired "first 40 characters"
  heuristic).
- Specific keyword orderings in the title beyond "type + brand + model + spec".
- Whether a secondary synonym in the description measurably helps.
- Impact of description length / structure on conversion for a given category.
- Effect of image count or gallery composition on ranking (vs. only on conversion).

## Do not

- Invent ranking factors ML doesn't document ("ML boosts listings with 8 images",
  "keyword density of X%", "the first N characters carry more ranking weight",
  "repeating keywords in the description ranks higher") — EXPERIMENTAL at best,
  never stated as OFFICIAL. ML frames the description as *complementing*
  structured product information, not as a ranking field (⚠ verify).
- State an absolute visibility claim ("your listing will not be shown") as
  OFFICIAL unless ML documents that exact behavior.
- Keyword-stuff the title or attributes. It violates title rules and risks review.

## Audit checks (`SEARCH_RELEVANCE`)

- [ ] Category resolved **and** validated (leaf, `listing_allowed`, domain fits) —
      see `categories.md` §1, not merely predicted.
- [ ] Clear product identity; the most product-defining terms lead the title /
      `family_name`.
- [ ] All relevant, evidence-backed structured attributes are filled.
- [ ] Marketplace / customer terminology used; no irrelevant terms; no stuffing.
- [ ] Brand / model spelling consistent everywhere (where they legitimately exist).
- [ ] Product identifier present with provenance, or legitimate absence handled
      via `EMPTY_GTIN_REASON` (`attributes.md`).
- [ ] Title / `family_name` consistent with the product and its attributes.
- [ ] No undocumented ranking factor (first-N-characters, keyword density,
      image-count-for-rank) treated as OFFICIAL; every relevance claim tagged
      OFFICIAL / INTERNAL / EXPERIMENTAL.

## Sources

- Como criar anúncios eficientes — https://vendedores.mercadolivre.com.br/nota/como-criar-anuncios-eficientes-no-mercado-livre — Central de Vendedores — 2026 ⚠ verify — consulted 2026-08-27 — complete/high-quality listing framed as better for exposure.
- O status das suas fichas técnicas — https://vendedores.mercadolivre.com.br/notas/o-status-das-suas-fichas-tecnicas/ — Central de Vendedores — 2026 ⚠ verify — consulted 2026-08-27 — incomplete fichas lose relevance / go to review; fichas feed filters. Does **not** state an absolute "not shown" rule.
- Identificadores de produtos — https://developers.mercadolivre.com.br/pt_br/identificadores-de-produtos — Developers — ⚠ verify — consulted 2026-08-27 — universal identifier helps catalog matching.
- "First N characters / first words weigh more", "keyword density", "description repetition ranks": third-party SEO material only (tray.com.br, base.com, gosmarter.com.br) — EXPERIMENTAL, never OFFICIAL.
