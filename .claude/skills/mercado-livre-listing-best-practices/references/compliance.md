# Compliance & marketplace policy

last_reviewed: 2026-08-27
volatile: true

Marketplace-**policy** risk — conceptually different from SEO, image quality,
category choice or ProductMaster completeness. This file defines *how* Product
Factory handles compliance evidence and dynamic policy checks; it does **not**
reproduce Mercado Livre's full policy catalogue (policies change — resolve them
dynamically).

## 1. Three separate questions — never collapsed

| Axis | Question | Owner |
|---|---|---|
| **Evidence** | *Is this claim true?* (`CONFIRMED` / `INFERRED` / `MISSING` / `CONFLICTING` / `UNSUPPORTED`) | `attributes.md` §evidence |
| **Compliance** | *May this claim / product be published here?* | this file |
| **Quality** | *Is this the best way to communicate it?* | `quality-audit.md` |

A claim can be factually true but prohibited in a given marketplace context. Do
not merge truthfulness with marketplace permission.

## 2. ComplianceFinding (procedural — not a DB schema)

```
ComplianceFinding {
  rule                # what policy / requirement
  source              # ML page / API / INTERNAL
  classification      # OFFICIAL | DYNAMIC | INTERNAL | UNVERIFIED
  scope               # product type / category / claim / media / seller
  evidence            # what supports or refutes applicability
  status              # PASS | REVIEW | FAIL
  affects[]           # CONTENT | PUBLICATION | EXECUTION  (one finding may touch several — see §8)
  remedy?             # the action that resolves it
}
```

`affects` is an **array** and mirrors the general finding field (`SKILL.md` §9).

Compliance is **not** a fifth readiness status — findings feed the existing
dimensions (`SKILL.md` §8).

## 3. Prohibited / restricted / regulated — resolve dynamically

```
product type → policy resolution → allowed | restricted/conditional | prohibited | unresolved
```

| Result | Meaning | Effect |
|---|---|---|
| `allowed` | No known prohibition from the executed policy checks. | none |
| `restricted / conditional` | Publication needs a specific category, authorization, certification, seller eligibility, special attributes, or legal conditions. | **REVIEW** until satisfied; **PUBLICATION_STATUS = FAIL** once a condition is confirmed applicable and unmet. |
| `prohibited` | A confirmed policy forbids sale/publication. | **PUBLICATION_STATUS = FAIL** (and **EXECUTION_STATUS = FAIL** for an attempted publish). |
| `unresolved` | Policy check not executed / evidence insufficient, **and** the product category reasonably requires a determination. | **REVIEW**. |

A confirmed **prohibited** product may still have **`CONTENT_STATUS = PASS`** —
its product facts can be fully known and its description truthful. The prohibition
is a *marketplace-permission* failure, not a *product-truth* failure. Primary
impact: `PUBLICATION_STATUS = FAIL`, and for an attempted publish
`EXECUTION_STATUS = FAIL`. Only fail `CONTENT_STATUS` if the content *itself*
carries a truth / evidence problem (e.g. it asserts an unsupported authenticity
or authorisation claim).

- Do **not** hardcode a giant permanent prohibited list — cite the live policy
  pages and resolve per product. (OFFICIAL policy surfaces: "Produtos proibidos",
  "Práticas proibidas", IP-infringing content, stolen/off-market goods, ANVISA
  compliance — see Sources.)
- Do **not** mark every ordinary fashion/accessory product `unresolved`/REVIEW
  just because a policy check could theoretically exist — use risk and context.
- Known-prohibited examples from current OFFICIAL policy (⚠ verify — illustrative,
  not exhaustive): medicines of any kind (except the official ML Farma channel),
  tobacco / e-cigarettes, cannabis and derivatives, counterfeits, goods
  prohibited by law, stolen property.

## 4. Regulated products — connect to CONDITIONAL_REQUIRED

Certifications / regulatory info / warranties / compatibility are
`CONDITIONAL_REQUIRED` (`SKILL.md` §2 B). Compliance decides applicability;
evidence decides the value.

```
requirement applicability unresolved      → REVIEW
confirmed applicable + evidence missing   → PUBLICATION_STATUS = FAIL
confirmed applicable + evidence present   → fill from CONFIRMED evidence
```

- Never infer *which* certification applies from a generic product type alone
  (e.g. do not assume ANATEL / INMETRO / ANVISA just from "electronics" or
  "cosmetic"). Applicability must be resolved (category, regulation, product
  facts), then evidence supplied.
- Never invent a certification, registration number, or approval body.

## 5. Claims safety

Product Factory may publish a **material claim only when evidence supports it**.
Watch words: "original", "official", "genuine", "authentic", "licensed",
"authorized", "certified", "approved by", "medical", "therapeutic", "hypo-
allergenic", "waterproof", "100% protection", "gold" / "silver" (vs plated /
coloured), material composition, compatibility, warranty terms, performance
numbers.

```
claim unsupported AND removable        → drop the claim; CONTENT_STATUS may stay PASS
claim unsupported AND essential to the
  product identity / sale              → REVIEW or CONTENT_STATUS = FAIL (by evidence)
```

Evidence stays `UNSUPPORTED` / `MISSING` per existing semantics — do not
"resolve" a claim by inventing backing. Quality optimisation **cannot** reduce
evidence integrity (§9).

## 6. Brand, authenticity & intellectual property

- Do not claim *genuine / authentic / original / licensed / authorized* without
  evidence. Acceptable evidence: manufacturer/supplier documentation, invoice,
  written authorization, legitimate packaging/markings, ML catalog identity.
  Weak evidence is **not** conclusive proof of authenticity → REVIEW.
- Do not use another brand for traffic; do not imply **compatibility = affiliation**.
  Distinguish: product fact · brand/trademark use · compatibility reference ·
  copyrighted media. Flag questionable use for human review.
- **Brand Protection Program (BPP)** complaints and existing brand-protection
  moderations are **external compliance signals**, not proof the seller violated
  policy — treat a complaint as a **high-priority compliance REVIEW**; a
  confirmed marketplace action affects **EXECUTION_STATUS**. A seller response
  may require real evidence (a licence document). Do not build a dispute workflow
  here.
- The Skill does not adjudicate IP law — it uses marketplace policy + seller/
  product evidence and escalates the rest.

## 7. Contact information / external diversion

Where confirmed by current ML moderation policy, listing content must not carry
phone / WhatsApp / e-mail / external website / social profile / off-marketplace
payment / external purchase CTA / QR codes. (`descriptions.md` and `images.md`
already forbid this in their surfaces — this is the policy anchor.) Do not
universalise exact prohibited formats without current official evidence.

**Readiness impact.** Such a string is a *removable* content-policy violation, not
a product-truth failure — the product facts are still known:

```
prohibited contact / diversion string detected
        ↓
remove it from the generated content
        ↓
CONTENT_STATUS may remain PASS   +   record a compliance/content finding
        ↓
if the assembled payload still contains it at publish time
        ↓
PUBLICATION_STATUS = FAIL   (optionally QUALITY_STATUS = REVIEW while pending)
```

Do **not** treat a removable policy string as a `CONTENT_STATUS` failure — that
is reserved for a genuine truth / evidence problem (core identity unknown,
material fact conflict, essential claim needs fabrication, the content represents
a different product, Product Identity Guard failure).

## 8. Moderation — post-publication marketplace state

Publication success is **not** permanent compliance. A listing can later be
moderated: `active` with exposure loss, `paused`/reviewable, or `inactive`/
non-recoverable. Read state from the moderation APIs — never assume "no
moderation found" means a new payload is compliant (absence of enforcement ≠
approval). A successful `/items/validate` and a successful publication are also
not compliance approval.

**Endpoints** (verified 2026-08-28, search-indexed; live 403):

- `GET /moderations/last_moderation/{MODERATION_REFERENCE_ID}` — the last
  moderation for one element. `MODERATION_REFERENCE_ID = <element_id>-<element_type>`
  (suffix `-ITM` for a listing; `-QUE` question, `-REV` review). **Not**
  `GET /moderations/last_moderation` on its own.
- `GET /moderations/infractions/{USER_ID}` — the seller's infraction history
  (query params: `related_item_id`, `element_id`, `element_type`, date range,
  language, pagination, sort). **Not** a top-level `GET /infractions`.

```
ModerationFinding {
  type              # moderation group (contact/links, brand protection,
                    #   prohibited product, image quality, incomplete_technical_specs,
                    #   catalog restriction, vertical-specific, …)
  item_status       # the Item's own state: active | paused | inactive
  infraction_state  # forbidden (final) | waiting_for_patch | held | pending_documentation (temporary)
  reason            # ML text (HTML) — preserve verbatim
  remedy            # ML text — only present when recoverable; preserve; do not fabricate
  recoverable?      # derive from presence of remedy + infraction_state
  source            # /moderations/last_moderation/{ref} | /moderations/infractions/{user}
  observed_at
}
```

**Item state ≠ moderation/infraction state.** `active` / `paused` / `inactive`
describe the *Item*; `forbidden` / `waiting_for_patch` / `held` /
`pending_documentation` describe the *infraction*. A moderation finding can drive
consequences for the Item, but they are separate fields. Preserve **both**
`reason` and `remedy` — never reduce moderation to `moderated = true`; where
`remedy` is absent for a non-recoverable condition, do not invent one.

| Observed state | Typical mapping (inspect reason/remedy first — do not apply blindly) |
|---|---|
| `active` with exposure loss | `QUALITY_STATUS = REVIEW` (and compliance REVIEW if the reason is policy) |
| `paused` / recoverable | `EXECUTION_STATUS = REVIEW` or `FAIL`, per the operation and remedy |
| `inactive` / non-recoverable | `EXECUTION_STATUS = FAIL` for any operation needing that listing |
| `incomplete_technical_specs` | `QUALITY_STATUS = REVIEW` — a ranking penalty, **not** a publication block (§ attribute semantics, `quality-audit.md`) |
| image-quality (`poor_quality_thumbnail`) | Correction 05 owns image policy — hard violation → PUBLICATION/EXECUTION as appropriate; quality-only → `QUALITY_STATUS = REVIEW`; identity distortion → Product Identity Guard `IDENTITY_FAIL` |

**Remedy execution rule:** if a remedy asks to add/correct information, use
verified ProductMaster facts / marketplace state / seller evidence — **never
fabricate a fact to reactivate a listing**. Missing remedy fact → REVIEW +
human evidence request. Do not recreate/duplicate the listing as a workaround.

**Marketplace moderation does not define ProductMaster truth.** If ML suspects
e.g. the wrong brand: preserve ProductMaster evidence, record the marketplace
finding, reconcile evidence, and update ProductMaster only if evidence justifies
it.

## 9. Quality optimisation is subordinate to truth

Never, to improve a quality score / completeness: add unsupported specs, add a
fake GTIN, add guessed dimensions, change product colour, claim a certification,
or associate to a catalog product merely for exposure. A `/performance`
recommendation to "add attribute X" does **not** authorise inventing X's value —
evidence is still required.

## 10. Not compliance evidence

- **Competitor listings** — if competitors use prohibited claims, show logos,
  omit required info, or sell questionable products, that is **not** permission.
  `competitor presence ≠ marketplace permission`.
- **Buyer reviews** — reveal expectation mismatch / quality complaints (feeds
  quality + return-prevention), but do **not** establish regulatory approval,
  authenticity, certification, or marketplace permission.

## 11. INTERNAL high-risk claim routing (label INTERNAL — not a prohibition list)

Trigger stronger evidence + compliance review, do not by themselves declare a
claim prohibited: health · safety · certification / legal approval ·
authenticity · extreme performance · child safety · regulated compatibility.

Illustrative (INTERNAL evidence/compliance examples — not ML policy text):
eyewear "blocks blue light" needs lens evidence and must not become "protects
eyesight"; a gold-coloured alloy is "gold-plated / golden finish", not "gold",
unless plating/material evidence supports it; never invent ANATEL for
electronics; never claim vehicle compatibility without evidence.

## Rule classification for compliance

Be strict about provenance. Never turn seller folklore, competitor behaviour,
forum advice or an internal heuristic into an `OFFICIAL` compliance rule.
A learned pattern ("listings with X are often moderated") is `LEARNED` → may
trigger REVIEW; it never creates an OFFICIAL FAIL on its own.

## Sources

- Produtos proibidos — https://www.mercadolivre.com.br/ajuda/Produtos-proibidos_1029 — Central de Ajuda — ⚠ verify (search-indexed 2026-08-27) — prohibited-product categories; non-compliance → listing removal + account penalties.
- Práticas proibidas para vender — https://www.mercadolivre.com.br/ajuda/contornar_estrutura_programas_4822 — Central de Ajuda — ⚠ verify (search-indexed 2026-08-27).
- Conteúdos que infrinjam a propriedade intelectual — https://www.mercadolivre.com.br/ajuda/1078 ; Propriedade roubada / Produtos fora do comércio — https://www.mercadolivre.com.br/ajuda/Propriedade-roubada_1034 — Central de Ajuda — ⚠ verify (search-indexed 2026-08-27).
- Como cumprir as normas da ANVISA — https://vendedores.mercadolivre.com.br/nota/como-cumprir-as-normas-da-anvisa-e-evitar-o-cancelamento-do-seu-anuncio — Central de Vendedores — ⚠ verify (search-indexed 2026-08-27) — regulated-product compliance example.
- Brand Protection Program — https://www.mercadolivre.com.br/ajuda/Programa-de-Prote%C3%A7ao-Propriedade-Intelectual_2099 , https://www.mercadolivre.com.br/brandprotection/enforcement — ⚠ verify (search-indexed 2026-08-27) — rights-holder report + seller response (4 calendar days; licence doc PDF/PNG/JPG ≤ 5 MB); no reply → auto-removal.
- Manage moderations — https://developers.mercadolivre.com.br/en_us/manage-moderations — Developers — verified 2026-08-28 (search-indexed; live 403) — `GET /moderations/last_moderation/{MODERATION_REFERENCE_ID}` (`<element_id>-<element_type>`, suffix `-ITM` / `-QUE` / `-REV`); `GET /moderations/infractions/{USER_ID}` (query: `related_item_id`, `element_id`, `element_type`, dates, language, pagination, sort). `reason` (HTML) always; `remedy` (HTML) only when recoverable. Infraction states `forbidden` (final) / `waiting_for_patch` / `held` / `pending_documentation` (temporary) — distinct from the Item's `active` / `paused` / `inactive`. ML also pauses preventively (unusual price change, items without sales, image-by-URL not yet processed).
- Qualidade das publicações / listing `/performance` — https://developers.mercadolivre.com.br/pt_br/qualidade-das-publicacoes — Developers — verified 2026-08-27 (search-indexed; live 403) — **`/health` discontinued, replaced by `GET /item/$ITEM_ID/performance`**; response `level_wording` (per site), entities `mode` = `OPPORTUNITY` (improve) or `WARNING` (issue reducing score).
- Catalog required listings / `catalog_only_restricted` — https://developers.mercadolivre.com.br/pt_br/publicacoes-necessarias-do-catalogo , https://developers.mercadolivre.com.br/en_us/catalog-eligibility — Developers — verified 2026-08-27 (search-indexed; live 403) — catalog-exclusive domains: marketplace publication moderated `under_review` with `catalog_only_restricted`; catalog-required: `catalog_listing_eligible` + product `listing_strategy: catalog_required` → moderated `opt_obey`; applies to MLB.
- Validador de publicações (`POST /items/validate`) — see `official-sources.md` — a pre-publication technical check, **not** a compliance certification or moderation pre-clearance.
- SELLER_PACKAGE_* / ME2 — see `official-sources.md` — package dimensions can be mandatory by shipping mode (ME2) without appearing as category `required`; missing → moderated / not published (PUBLICATION_STATUS).
