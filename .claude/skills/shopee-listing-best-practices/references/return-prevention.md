# Return prevention

last_reviewed: 2026-08-28
classification: INTERNAL · SHARED_CORE_CANDIDATE (built on the expectation that a listing must faithfully represent the product)

Purpose: shrink the gap between what the listing makes a Shopee buyer expect and
what arrives. Run before finishing every listing.

## Mandatory question

> **"Could a reasonable buyer interpret this listing as a materially different
> product from what they will actually receive?"**

If yes → produce a specific correction (which surface, what change) and raise a
finding. An unanswered "yes" → **BLOCKER** (`affects: [CONTENT]`, and
`[PUBLICATION]` where the mismatch is also a policy / moderation risk —
`references/compliance.md`).

## Pre-finish checklist

Verify each is stated **correctly and unambiguously** across attributes,
description, images and the variant table:

- [ ] **Size / dimensions** — real measurements; units; a dimension image if size
      is a common surprise.
- [ ] **Colour** — matches the real product and the specific model; images not
      over-saturated; note that colour may vary with the screen if relevant.
- [ ] **Material** — the actual material; no upgrade by wishful wording.
- [ ] **Model / version / edition** — exact; no implying a newer / higher model.
- [ ] **Quantity / kit contents** — units per pack; single or set? pair or one?
- [ ] **What's in the box / what's NOT** — every included item listed; batteries,
      charger, mounting kit, app subscription, SIM, etc. called out explicitly.
- [ ] **Accessories shown in photos** — anything not included labelled
      "ilustrativo / não incluso".
- [ ] **Compatibility (fitment)** — declared clearly; caveats stated.
- [ ] **Installation / assembly** — required? tools included? difficulty?
- [ ] **Usage / limitations** — water resistance, weight limit, voltage,
      indoor-only, etc.

## Shopee-specific hotspots (research hypotheses — not an exhaustive rule set)

Phase 01 flags these as likely mismatch areas on Shopee; corroborated by Shopee
return reasons, not by an explicit Shopee rule list:

- **Model / variation confusion** — ambiguous tier-option names or model images
  → the buyer selects the wrong model. Each model's option name + image + price +
  stock must be unambiguous and correct.
- **Quantity / kit** contents.
- **Size & measurement** vs a size chart.
- **Compatibility / fitment.**
- **Material / colour rendering** under Shopee's image compression.
- **"What's in the box".**

## Output

A `return_prevention` block: checklist result, the answer to the mandatory
question, and each correction with its target surface and severity.

## Sources

- Built on the expectation that a listing must faithfully represent the product
  (`seller.shopee.com.br/edu` — Centro de Educação — consulted 2026-08-28 —
  `SEARCH_INDEXED`) and corroborated by Shopee return reasons.
- Test + checklist shape — `.claude/skills/mercado-livre-listing-best-practices/references/return-prevention.md`
  — internal — architectural reference only. The checklist itself is INTERNAL.
- Hotspots: `research/shopee-listing-skill/discovery-report.md` §23, §"Return
  Prevention (brief §42 — detail)".
