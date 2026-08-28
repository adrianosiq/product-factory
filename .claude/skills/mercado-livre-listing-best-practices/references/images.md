# Images

last_reviewed: 2026-08-27
source_last_updated: 2026-03-24 (Trabalhar com imagens, per requester) — ⚠ verify
volatile: true

Three separate layers — **never merge them into one OFFICIAL block**:

- **A. OFFICIAL** — Mercado Livre rules / behaviour from ML documentation & API.
- **B. INTERNAL commercial gallery strategy** — Product Factory practice to
  improve comprehension and conversion; never an ML rule.
- **C. Product Identity Guard** — Product Factory integrity for generated / edited
  images: *presentation may change, product identity may not.*

---

## A. OFFICIAL Mercado Livre image rules

### A.1 Technical specs — Developers "Trabalhar com imagens"

Verified 2026-08-27 against a search-indexed copy; the live page returns 403 to
bots, so keep `⚠ verify` until a live read.

| Item | Value | Kind |
|---|---|---|
| Accepted formats | JPG, JPEG, PNG | REQUIRED (API rejects other formats) — ⚠ verify |
| Colour space | RGB preferred over CMYK | RECOMMENDED — not a documented hard reject |
| Recommended resolution | 1200 × 1200 px (larger is resized down to fit) | RECOMMENDED + TECHNICAL BEHAVIOR |
| Accepted minimum | 500 × 500 px ("version M") | REQUIRED (accepted minimum) — ⚠ verify |
| Accepted maximum | 1920 × 1920 px ("version F") | TECHNICAL BEHAVIOR (resized) — ⚠ verify |
| Max file size | 10 MB per image, on `POST /pictures/items/upload` | REQUIRED for that upload endpoint — ⚠ verify |
| Product occupation | subject fills ~95% of the frame | RECOMMENDED |
| Zoom | image width > ~800 px activates the hover-zoom widget | TECHNICAL BEHAVIOR — not a publish gate |
| smartcrop | optional pre-send check that trims excess background for a good product-to-frame ratio | TECHNICAL BEHAVIOR |

- Do **not** turn a RECOMMENDED value (1200 × 1200, ~95 %, RGB) into a publish
  FAIL. A hard FAIL applies only to a rule the API / moderation actually enforces:
  unsupported format, below the accepted minimum, over the size cap, or over the
  resolved category max count (A.2).
- Anywhere the skill lists formats, only **JPG / JPEG / PNG**. No GIF, WEBP, TIFF,
  BMP or SVG — do not infer support because a URL can serve the bytes.

### A.2 Image count — DYNAMIC, per category

- Max per item: `settings.max_pictures_per_item`. Max per variation:
  `settings.max_pictures_per_item_var`. Both read from
  `GET /categories/$CATEGORY_ID`. **Never hardcode 6 / 8 / 10 / 12.**
- Some categories / verticals also set a **minimum** count (e.g. fashion "Tops":
  min 4). Minimums are DYNAMIC / category-specific too.
- Legacy or general ML pages that say "up to 6 images" are superseded by the
  category configuration — use the resolved category value and note any
  discrepancy under `Still unverified`.

### A.3 Main image / cover photo — OFFICIAL, category-dependent

ML seller "Fotos de qualidade…" + Central de Ajuda "Como tirar boas fotos dos
seus produtos" — verified 2026-08-27 via search-indexed copies; `⚠ verify`.

- **Prohibited on the cover photo** (a top cause of automatic moderation
  pausing): store logos, watermarks, promotional text / price / discount badges,
  shipping claims, contact info, URLs, social handles, QR codes, decorative
  borders / frames.
- **White background**: *strictly required* in some categories (e.g. Tecnologia,
  Beleza, Saúde, Supermercado — pure white). Other categories (e.g. Moda, Casa &
  Móveis) permit neutral backgrounds or in-context images. **Resolve the rule for
  the actual category** — do not assume a universal, Amazon-style pure-white
  policy.
- Cover photo shows the product only (no props / hangers), well lit, no hard
  shadows.

### A.4 Moderation & tooling — OFFICIAL (⚠ verify)

- ML moderates listing images for quality; a flagged listing shows `active` /
  `paused` with tag `poor_quality_thumbnail` (see ML "Manage moderations").
- Upload: `POST /pictures/items/upload` (multipart) to ML's CDN.
- **Image Diagnostics API** accepts base64 / `picture_id` / URL; prioritise the
  main image (thumbnail).
- A local Product Factory pass **never guarantees** ML acceptance — marketplace
  moderation stays authoritative. These endpoints are noted as future execution
  mechanisms; MCP wiring is out of scope here.

### A.5 Required vs optional images — DYNAMIC

Whether images are mandatory, and the minimum count, depends on listing type /
category / vertical / current API validation (`requires_picture` where exposed).
Model as DYNAMIC. A category-specific rule (e.g. real estate) is **not** a
universal product-listing rule.

---

## B. INTERNAL — commercial gallery strategy (never an ML rule)

- **Target ≈ 8–10 *useful* images when the category max allows** — a guideline,
  not a quota and not an ML recommendation. Never add filler to hit a number.
  `useful coverage > image count`: a simple product may need fewer; a complex one
  more, within the category max.
- Build the gallery from real product characteristics, evidence, customer
  objections, variant structure and marketplace constraints. Never fabricate a
  benefit to fill a slot.

Conceptual coverage (skip what doesn't apply — **not** a fixed 10-slot template):

| Role | Purpose |
|---|---|
| Primary / hero | Immediately recognisable product; clean, faithful colour, correct variant. |
| Alternate angle | Remove shape / back doubt. |
| Close-up / material / finish | Texture, seams, ports, controls. |
| Dimensions / scale | Measured drawing or product beside a known reference. |
| Key benefit / feature | The #1 reason to buy, shown. |
| Usage / context | Real use environment. |
| Objection-handling | Address the top recurring complaint / question. |
| Variant differentiation | The options, each labelled, matching the picker. |
| Packaging / what's included | Everything included; note what is NOT. |
| Extra detail | Where genuinely useful. |

- **Hero strategy (INTERNAL)**: clean neutral / white background, minimal
  distraction, high product visibility, faithful colour, category-appropriate
  framing. Where ML's category policy is stronger (A.3), the OFFICIAL rule wins.
- **Photographic-production presets** — exact angle (e.g. 15°), exact frame
  occupancy (e.g. 85 %), lighting direction, lens / focal length ("85 mm look") —
  are creative templates, **not** marketplace rules and **not** audit criteria.
  Any `85%` / `15°` figure in the skill is an INTERNAL preset, never ML policy,
  and is distinct from ML's ~95 % occupancy recommendation (A.1).
- **Product-type patterns** (eyewear: front / three-quarter / temple / hinge /
  frame detail / dimensions / face context where allowed; jewellery: front /
  side / clasp / stone-and-finish detail / scale / lifestyle) are INTERNAL
  planning patterns, not requirements.
- **Lifestyle images**: allowed as INTERNAL tools for scale, context, use, desire
  and objection handling. They must not imply non-included accessories, change
  product appearance, exaggerate size, invent functionality, or confuse what is
  being sold. Contextual objects must be visually distinguishable from what is
  included when it matters. ML does not require lifestyle images.
- **Infographic / dimension / callout images**: fine when evidence supports the
  content and the category permits text / graphics. Never show unsupported
  dimensions, invented specs, fake certification badges or fabricated comparison
  data. Apply any category limit on in-image text / graphics.

---

## C. Product Identity Guard (Product Factory integrity)

**Presentation may change. Product identity may not.** Applies to every generated
or edited asset — AI generation, AI editing, retouching, background replacement,
lifestyle generation, relighting, compositing, upscaling, angle reconstruction,
variant visualisation. ML may technically accept an AI image; Product Factory
still rejects it if identity is distorted.

### C.1 Protected properties (adapt to the product — not an exhaustive schema)

Unless evidence explicitly supports the change, generation / editing must **not**
alter: geometry, shape, proportions, dimensions, silhouette; the number and
location of components; hardware, closures, hinges, lenses, stones, pearls,
buttons, ports, seams; texture, material appearance, finish; colour,
transparency, gloss, metallic finish, pattern; branding, logos, printed
markings; included accessories; packaging contents.

### C.2 Colour fidelity

Product colour is identity data. Do not casually shift hue, saturation,
transparency, metallic / gloss finish, stone colour, lens tint, frame colour or
printed pattern. Lighting may affect appearance, but the output must stay
representative of the real unit. Colour not confidently preservable → REVIEW.
Never invent a new sellable colour variant from imagination.

### C.3 Geometry fidelity

Do not lengthen, shorten, widen, narrow, reshape, symmetrise, "beautify",
straighten or simplify the product in any way that changes what the buyer
receives — especially eyewear, jewellery, fashion accessories, footwear, parts,
electronics, fitted goods. Photographic perspective changes are acceptable only
when they do not misrepresent the physical product.

### C.4 Material fidelity

No visual upgrades: plastic → metal, alloy → precious metal, synthetic → natural
pearl, glass → crystal, resin → gemstone, plated → solid gold, generic plastic →
carbon fibre. Appearance must not imply an unsupported material claim.

### C.5 Component fidelity

Do not add or remove accessories, screws, stones, decorative pieces, hinges,
lenses, cases, cables, adapters, straps, packaging items or replacement parts.
In a lifestyle composition, contextual objects must be visually distinguishable
from what is included in the sale when that matters.

### C.6 Defect / condition fidelity

When the image represents the actual sellable unit, do not edit out a relevant
real condition (scratches, wear, dents, missing pieces, discoloration, defects,
used-condition evidence). For new generic inventory, cleaning up photographic
dust / background artefacts unrelated to the product is acceptable. Never turn a
used or damaged unit into a visually new product.

### C.7 Scale fidelity

Lifestyle / generated images must not create misleading scale (on a person, in a
room, beside an object, in a hand). Relative scale must be credible and backed by
known dimensions where scale matters. Unknown dimensions → no precise scale claim
through imagery; use neutral context or mark the asset for review.

### C.8 Variant identity

A variant image must depict the actual variant. Do not generate a "red variant"
from a black source image unless the red variant is independently confirmed and
the transformation can faithfully represent it. A variant image must not
introduce different geometry, components, material or pattern unless those
differences really belong to that variant. Detailed User Products picture
mechanics remain Correction 06.

### C.9 Evidence model

- Every image traces to product evidence: supplier / seller / manufacturer
  photos, verified ProductMaster attributes, confirmed dimensions, confirmed
  variant info, packaging evidence.
- **A generated / edited image is an output, not evidence.** It cannot become
  independent evidence for a fact the generation itself introduced — an AI-drawn
  zipper not visible or confirmed in the source does **not** prove the product
  has a zipper. No circular evidence.
- Image generation must not compensate for missing product evidence. If the
  source does not show enough to preserve identity (hidden side, unknown back /
  hinge / clasp / texture / packaging / variant colour), record an evidence gap —
  prefer `MISSING` / REVIEW over plausible invention.
- Image generation consumes verified ProductMaster + source images + variant
  data + confirmed dimensions / materials + the image plan. It never produces new
  ProductMaster facts.

### C.10 Transformation risk classes (INTERNAL — not ML policy)

| Class | Examples | Handling |
|---|---|---|
| **LOW-RISK** presentation | background cleanup / replacement, crop, exposure & white-balance, resolution upscale, shadow cleanup, controlled relighting | allowed with a light identity check |
| **MEDIUM-RISK** | lifestyle compositing, contextual placement, angle reconstruction, model / person placement, scale demonstration | stronger review + evidence |
| **HIGH-RISK / prohibited without direct evidence** | product redesign, geometry alteration, material substitution, colour invention, component invention, feature invention, logo / brand invention, false packaging, false included accessories | rejected unless evidence explicitly supports it |

### C.11 Post-generation identity audit

Compare every generated / edited asset against source images, ProductMaster,
variant identity, known dimensions, materials and components. Procedural result
(not a change to the global status model):

- `IDENTITY_PASS` — no material product-identity deviation.
- `IDENTITY_REVIEW` — possible deviation, or source evidence insufficient to
  verify a generated detail.
- `IDENTITY_FAIL` — confirmed material alteration / misrepresentation.

`ImagePlan` (requested roles) is separate from `ImageAsset` (the produced file):
every asset must still pass marketplace constraints (A), the evidence model and
the Guard (C). A commercial goal never overrides product truth.

---

## Per-variant images

- Map each variant to its own image set (respecting the resolved
  `max_pictures_per_item_var`).
- The hero of each variant shows **that** variant.
- Do not reuse one colour's photos for another colour.

## Audit checks (`IMAGES`)

**OFFICIAL (A):**
- [ ] Format is JPG / JPEG / PNG; file ≤ 10 MB; resolution ≥ the accepted minimum.
- [ ] Count ≤ the resolved category `max_pictures_per_item` / `_var`, and ≥ any
      category minimum.
- [ ] Cover photo: product only; no logo / watermark / promo text / badge /
      border / QR / contact info; background rule correct **for this category**.
- [ ] Recommended-only values (1200 × 1200, ~95 %, RGB) met where practical — a
      miss is a WARNING, not a FAIL.

**Identity (C):**
- [ ] Every asset traces to product evidence; no generated image used as evidence
      for a fact it introduced.
- [ ] Generated / edited assets pass the post-generation identity audit
      (`IDENTITY_PASS`); `IDENTITY_REVIEW` recorded; `IDENTITY_FAIL` blocks.
- [ ] Colour, geometry, material, components, condition and scale faithful to the
      real unit.
- [ ] Variant images depict the correct variant.

**Commercial (B):**
- [ ] Each image has a real job; no filler. Gallery gaps → WARNING / score, not a
      publish FAIL (unless a category minimum or a critical comprehension gap).
- [ ] Rights to every image are clear (no competitor / unlicensed stock).

## Sources

- Trabalhar com imagens / Working with pictures — https://developers.mercadolivre.com.br/pt_br/trabalhar-com-imagens , https://developers.mercadolivre.com.br/en_us/working-with-pictures — Developers — updated 2026-03-24 (per requester); verified 2026-08-27 against a search-indexed copy (live page 403 to bots), keep `⚠ verify` — formats JPG/JPEG/PNG, RGB over CMYK, ~95 % occupancy, recommended 1200 × 1200, min 500 × 500 ("M"), max 1920 × 1920 ("F"), ≤ 10 MB, hover-zoom when width > ~800 px, smartcrop, `POST /pictures/items/upload`.
- Categories API image limits — `GET /categories/$CATEGORY_ID` `settings.max_pictures_per_item` / `settings.max_pictures_per_item_var` — Developers — corroborated 2026-08-27 (search-indexed; live 403) — per-category max image count for item and per variation; some categories/verticals also publish a **minimum** count.
- Image moderation / Manage moderations / Image Diagnostics — https://developers.mercadolibre.com.ar/en_us/manage-moderations , https://developers.mercadolibre.com.ar/en_us/image-diagnostics — Developers — verified 2026-08-27 (search-indexed; live 403) — image-quality moderation, `poor_quality_thumbnail` tag on `active`/`paused` listings; Image Diagnostics accepts base64 / `picture_id` / URL, prioritise the thumbnail.
- Fotos de qualidade: o segredo para se destacar e vender mais — https://vendedores.mercadolivre.com.br/nota/fotos-de-qualidade-o-segredo-para-se-destacar-e-vender-mais — Central de Vendedores — 2026 ⚠ verify (search-indexed 2026-08-27) — cover-photo rules, prohibited overlays, category-dependent white-background requirement.
- Como tirar boas fotos dos seus produtos — https://www.mercadolivre.com.br/ajuda/Como-tirar-boas-fotos-dos-seus-produtos_1320 — Central de Ajuda — ⚠ verify (search-indexed 2026-08-27) — cover photo product-only / white background / no promo overlays.
- Commercial gallery plan, hero strategy, product-type patterns, transformation risk classes, `IDENTITY_*` states: **INTERNAL** (Product Factory). Not ML rules.
