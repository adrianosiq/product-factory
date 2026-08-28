# SEO & discovery

last_reviewed: 2026-08-27

Mercado Livre does not publish a full ranking-factor list. Separate what ML
actually says from what is our strategy or a hypothesis.

## OFFICIAL (⚠ verify)

- **Category** is the primary routing signal — the listing only appears under the
  right category tree and its filters.
- **Attributes / ficha técnica feed the search filters.** If a buyer filters by a
  characteristic you left blank, your listing is **not shown** to that buyer.
- ML's search can match **keywords that appear in ficha-técnica fields**, not only
  the title.
- **Incomplete listings lose relevance** and may be sent to internal review until
  the missing fields are filled.
- ML frames "complete + high-quality listing" as better for exposure; and lists
  clear title, good photos and competitive commercial conditions as the levers of
  an "efficient" listing.
- **Universal code (GTIN/EAN)** helps match to catalog and improves findability.

## INTERNAL STRATEGY

- Put the most decisive, most-searched terms early in the title / `family_name`
  (see `titles-and-family-name.md`).
- Fill every attribute the ProductMaster supports, not just the required ones —
  each filled attribute is another filter you can appear in.
- Prefer a real structured attribute over free text in the description for the
  same fact (structured is filterable, description is not).
- Use the buyer's vocabulary (from `review-mining.md` / `competitor-research.md`)
  in the description and open attributes, without stuffing.
- Keep brand/model spelling identical across title, attributes and description so
  the same query matches all of them.

## EXPERIMENTAL (must be tested with performance data)

- Specific keyword orderings in the title beyond "brand+model+spec first".
- Whether adding a secondary synonym in the description measurably helps.
- Impact of description length / structure on conversion for a given category.
- Effect of image count or gallery composition on ranking (vs. only on conversion).

## Do not

- Invent ranking factors ML doesn't document ("ML boosts listings with 8 images",
  "keyword density of X%", etc.) — that's EXPERIMENTAL at best, and never stated
  as OFFICIAL.
- Keyword-stuff the title or attributes. It violates title rules and risks review.

## Audit checks (`SEARCH_RELEVANCE`)

- [ ] Category resolved **and** validated (leaf, `listing_allowed`, domain fits) — see `categories.md` §1, not merely predicted.
- [ ] All filterable attributes supported by the ProductMaster are filled.
- [ ] Decisive search terms present early in title/`family_name`.
- [ ] Brand/model spelling consistent everywhere.
- [ ] Product identifier present with provenance, or legitimate absence handled via `EMPTY_GTIN_REASON` (`attributes.md`).
- [ ] No stuffing; every relevance claim tagged OFFICIAL / INTERNAL / EXPERIMENTAL.

## Sources

- Como criar anúncios eficientes — https://vendedores.mercadolivre.com.br/nota/como-criar-anuncios-eficientes-no-mercado-livre — Central de Vendedores — 2026 ⚠ verify — consulted 2026-08-27.
- O status das suas fichas técnicas — https://vendedores.mercadolivre.com.br/notas/o-status-das-suas-fichas-tecnicas/ — Central de Vendedores — 2026 ⚠ verify — consulted 2026-08-27 — incomplete fichas lose relevance / go to review; fichas feed filters.
- Identificadores de produtos — https://developers.mercadolivre.com.br/pt_br/identificadores-de-produtos — Developers — ⚠ verify — consulted 2026-08-27 — universal code and catalog matching.
