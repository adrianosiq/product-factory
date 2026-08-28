# Return prevention

last_reviewed: 2026-08-27
classification: INTERNAL (return/complaint reduction), built on OFFICIAL accuracy expectations

Purpose: shrink the gap between what the listing makes a buyer expect and what
arrives. Run this before finishing every listing.

## Pre-finish checklist

Verify each item is stated **correctly and unambiguously** across attributes,
description and images:

- [ ] **Size / dimensions** — real measurements; units; a dimension image if size
      is a common surprise.
- [ ] **Color** — matches real product and the specific variant; images not
      over-saturated; note "cor pode variar conforme o monitor" if relevant.
- [ ] **Material** — the actual material (e.g. acetate vs TR90, leather vs PU);
      no upgrade by wishful wording.
- [ ] **Model / version / edition** — exact; no implying a newer/higher model.
- [ ] **Quantity** — units per pack; is it 1 or a set? Pair or single?
- [ ] **What's in the box** — every included item listed.
- [ ] **What's NOT included** — batteries, charger, mounting kit, app subscription,
      SIM, etc. stated explicitly.
- [ ] **Accessories shown in photos** — anything in a lifestyle shot that isn't
      included is labeled "ilustrativo / não incluso".
- [ ] **Compatibility** — declared via the proper mechanism; caveats stated
      ("não compatível com modelo X").
- [ ] **Installation / assembly** — required? tools included? difficulty?
- [ ] **Usage / limitations** — water resistance, weight limit, voltage,
      indoor-only, etc.
- [ ] **Variations** — each picker option maps to correct price/image/stock;
      differences explained.
- [ ] **Illustrative vs real** — renders/mock-ups clearly marked; hero is the
      real product.
- [ ] **Ambiguous phrasing** — no sentence that could be read two ways about scope,
      quantity or capability.

## Mandatory question

> **"Is there any reasonable way a buyer could interpret this listing as a
> different product from the one they will actually receive?"**

If yes → produce a specific correction (which surface, what change) and raise a
finding. A listing with an unanswered "yes" here → **BLOCKER**
(`affects: [CONTENT]`, and `[PUBLICATION]` where the mismatch is also a
policy/moderation risk — `compliance.md`). A material misleading omission is at
once a content, compliance and quality problem.

## Common failure patterns to catch

| Pattern | Fix |
|---|---|
| Lifestyle photo includes items not sold | Label "acessórios ilustrativos, não inclusos" + in-box image |
| "Kit" in title but only one unit | Fix title/quantity attribute |
| Material implied by look, not stated | State real material attribute |
| Photo color vs variant name mismatch | Re-map variant images |
| "Compatível com todos os modelos" overclaim | Use real compatibility list |
| Charger/battery assumed included | Explicit "não acompanha" line |
| Dimensions only in description, buyers miss them | Add dimension image + attribute |

## Output

A `return_prevention` block: checklist result, the answer to the mandatory
question, and each correction with its target surface and severity.

## Sources

- Built on the OFFICIAL expectation that the listing must faithfully represent the
  product (Central de Vendedores — "Como criar anúncios eficientes",
  https://vendedores.mercadolivre.com.br/nota/como-criar-anuncios-eficientes-no-mercado-livre — ⚠ verify — consulted 2026-08-27).
- Checklist itself is INTERNAL.
