# Quality audit & readiness

last_reviewed: 2026-08-27

Final gate. Produces the structured output in `SKILL.md` §9. No publishing.

## 1. Four readiness dimensions — independent

"Is this product valid?" is too broad. Answer four separate questions, each with
its own `PASS` / `REVIEW` / `FAIL`. They do **not** collapse into one status.

| Dimension | Question | Driven by |
|---|---|---|
| **`CONTENT_STATUS`** | Do we know enough about the real product to generate truthful content without inventing material facts? | ProductMaster + evidence |
| **`PUBLICATION_STATUS`** | Given the resolved ML context, does the listing meet the requirements to be publishable? | resolved category / attribute / GTIN / title-mode / image / variant / catalog / logistics requirements |
| **`EXECUTION_STATUS`** | Can Product Factory safely run the marketplace operation for this seller/account/context right now? | seller auth + tags, publication model, inventory mode, external mappings, marketplace/moderation state |
| **`QUALITY_STATUS`** | Beyond bare publishability, how complete and marketplace-optimised is the listing? | recommended attributes, technical-spec completeness, images, SEO, description, catalog match, `/performance` |

A product may legitimately be `CONTENT_STATUS = PASS` with the other three
`REVIEW` (facts are solid; API/seller/quality context not yet resolved) — that is
**not** a global FAIL. The reverse can also happen (API ready, product identity
unreliable → `CONTENT_STATUS = FAIL`, `EXECUTION_STATUS = PASS`).

### State meanings

- **`PASS`** — the dimension has sufficient verified information and no known blocker.
- **`REVIEW`** — unresolved dynamic context, missing non-blocking evidence,
  uncertainty, a human-judgment need, a marketplace check not yet executed, or an
  optimisation opportunity. Recoverable.
- **`FAIL`** — a known, **confirmed** condition makes the operation unsafe or
  invalid. FAIL requires evidence. **Never FAIL merely because something has not
  been checked** (Correction 02A): dynamic check pending → REVIEW; executed and
  confirms a mandatory incompatibility → FAIL.

## 2. Findings — severity vs status

`BLOCKER` and `WARNING` are **finding severities**, not status values.

| Severity | Meaning | Effect |
|---|---|---|
| **BLOCKER** | A confirmed condition that forces one or more listed dimensions to `FAIL`. Carries `affects: [ … ]`. | those dimension(s) = FAIL |
| **CRITICAL** | Serious gap, not a confirmed hard breach. | pushes the affected dimension to REVIEW |
| **WARNING** | Real but non-blocking (recommended attribute missing, weak framing, SEO opportunity, optional tech-spec incomplete). | normally `QUALITY_STATUS = REVIEW`; never forces CONTENT/PUBLICATION/EXECUTION FAIL |
| **RECOMMENDATION** | Improvement idea. | no status effect |

A blocker names its dimension(s). Some blockers are **cross-dimension** — e.g. a
confirmed counterfeit → `PUBLICATION_STATUS = FAIL` + `EXECUTION_STATUS = FAIL`
(attempted prohibited publish) + `CONTENT_STATUS = FAIL` if the copy would assert
authenticity. Do not force every finding into exactly one dimension; do document
which it affects.

## 3. Aggregation (deterministic, per dimension)

```
for each dimension:
  any applicable BLOCKER / confirmed FAIL   → FAIL
  else any applicable REVIEW / CRITICAL     → REVIEW
  else                                      → PASS
```

Warnings policy:

- `CONTENT_STATUS` / `PUBLICATION_STATUS` / `EXECUTION_STATUS` — warnings alone do
  **not** prevent `PASS`.
- `QUALITY_STATUS` — a **material** warning → `REVIEW`; minor/RECOMMENDATION → no
  effect. Not checking the marketplace quality state (`/performance`, moderation)
  is fine **pre-publication**; **post-publication**, an unchecked `/performance` /
  moderation state → `QUALITY_STATUS = REVIEW` until read.

### Derived compatibility `status` (not the source of truth)

Consumers should read the four dimensional statuses. A single `status` is kept
only for backward compatibility and is **derived**:

```
status = FAIL     if any of CONTENT/PUBLICATION/EXECUTION = FAIL
status = REVIEW   else if any dimension = REVIEW
status = PASS     otherwise
```

Label it *derived compatibility status* wherever it appears; new logic must not
depend on it.

## 4. Gates

| Gate | Requires | Notes |
|---|---|---|
| **Content generation** | `CONTENT_STATUS = PASS` | may proceed even if PUBLICATION/EXECUTION/QUALITY = REVIEW. Generate content before seller API context, stock location, final price or logistics are resolved, as long as product truth is sufficient. |
| **Publication** | `CONTENT_STATUS = PASS` **and** `PUBLICATION_STATUS = PASS` **and** `EXECUTION_STATUS = PASS` | `QUALITY_STATUS = REVIEW` may still be acceptable if the remaining issues are genuinely optional optimisation. |
| **Auto-publish (INTERNAL Product Factory gate — not an ML rule)** | the publication gate **and** no unresolved high-risk compliance REVIEW **and** the Product Factory approval policy satisfied | `QUALITY_STATUS` may be `PASS` or `REVIEW` per internal policy. Human approval can resolve REVIEW; it can **never** turn a confirmed OFFICIAL prohibition into "allowed". |

Internal quality thresholds are **INTERNAL policy**, never presented as ML
requirements.

## 5. `CONTENT_STATUS` — checks

`PASS` generally requires: product identity established; CORE_REQUIRED inputs
available (`SKILL.md` §2 A); relevant variant identities established; no
unresolved contradiction that materially affects content; content generable
without inventing product facts.

- **REVIEW** — a non-critical detail missing; weak-but-non-essential source
  evidence; an optional commercial detail unavailable; evidence ambiguity needing
  human confirmation while safe partial content is still possible.
- **FAIL** — core product identity or essential function unknown; conflicting
  evidence prevents determining what is actually being sold; a core variant
  identity cannot be established; content would require fabricating a material
  fact; an unsupported claim that is *essential* and cannot simply be omitted
  (`compliance.md` §5); Product Identity Guard `IDENTITY_FAIL` on an asset the
  content depends on.
- Missing **price / stock / warehouse / seller tag / listing type / an API
  response** does **not** cause `CONTENT_STATUS = FAIL` unless it genuinely
  affects content truthfulness.

## 6. `PUBLICATION_STATUS` — checks

`PASS` — all currently **resolved** mandatory publication requirements satisfied.

- **REVIEW** — category/attribute/API requirement still unresolved; conditional-
  attributes check pending; publication model pending; catalog requirement
  pending; required-image rule pending; a seller-specific publication requirement
  pending; logistics context unresolved where it could add mandatory
  requirements.
- **FAIL** — an **executed/resolved** rule confirms the payload/listing invalid:
  a resolved mandatory (`required`) attribute missing; category
  `settings.listing_allowed = false`; confirmed required identifier absent with
  no valid `EMPTY_GTIN_REASON`; a resolved `USER_PRODUCT` flow given `variations[]`
  or a forbidden manual `title`; an image over a **confirmed hard** limit or
  below the accepted minimum; a resolved catalog-only domain given an incompatible
  marketplace-only publication; a `/items/validate` confirmed **error** (not a
  warning); a confirmed prohibited product; a confirmed applicable regulated
  requirement with missing evidence; confirmed missing `SELLER_PACKAGE_*` for a
  resolved ME2 context.

`PUBLICATION_STATUS` is **pre-publication readiness** — do not reuse it to mean
"the item is currently active".

## 7. `EXECUTION_STATUS` — checks

`PASS` — required execution context resolved and compatible.

- **REVIEW** — seller/account or tags not loaded; publication model or inventory
  mode `UNRESOLVED`; warehouse/location mapping pending; the external
  `user_product_id` not yet returned; marketplace state must be refreshed; an
  async marketplace task still pending; a dynamic execution check not run.
- **FAIL** — a **resolved** execution attempt is incompatible or prohibited:
  Multi Origem account writing `/items` `available_quantity`; unauthorised /
  non-seller stock location; incompatible publication model; operation prohibited
  by current marketplace/moderation state; a needed API capability absent for the
  seller; a permanently-inactive listing required by the operation; an attempted
  publish of a confirmed prohibited product.
- Content generation must **not** depend on `EXECUTION_STATUS`.

## 8. `QUALITY_STATUS` — checks (the 0–100 scoring dimensions feed this)

`PASS` = no known material quality deficiency. `REVIEW` = a meaningful
optimisation opportunity exists, or marketplace quality state not checked
post-publication. `FAIL` = reserve for a quality defect severe enough to make the
listing misleading/unsafe, or a confirmed hard moderation state.

Score each 0–100; issues carry severity + affected dimension(s):

| Dimension | Pass bar | See |
|---|---|---|
| `PRODUCT_ACCURACY` | No invented facts; values CONFIRMED / vetted-INFERRED | `attributes.md` §evidence |
| `CATEGORY` | Resolved **and** validated (leaf, `listing_allowed`, domain + attributes fit) | `categories.md` |
| `CATALOG` | Catalog checked; link is exact match or independence justified | `catalog.md` |
| `FAMILY_NAME_TITLE` | Title mode detected; DYNAMIC length; brand/model only where legitimate & evidence-backed; no prohibited content; `family_name` = shared identity | `titles-and-family-name.md` |
| `ATTRIBUTES` | `required`/`new_required`/conditional filled; `value_id`s; identifier state valid, never invented; technical-spec completeness noted separately | `attributes.md`, `categories.md` |
| `SEARCH_RELEVANCE` | Clear identity; relevant structured attributes filled; real terminology; no stuffing; no undocumented-ranking folklore scored as fact | `seo-and-discovery.md` |
| `DESCRIPTION` | `plain_text` valid; complements ficha; no banned content; claims backed | `descriptions.md` |
| `IMAGES` | Confirmed hard specs met; count within resolved DYNAMIC max/min; cover-photo rules; recommended-only specs are WARNINGs; every asset evidence-backed & `IDENTITY_PASS`; variant-correct | `images.md` |
| `VARIANTS` | `PUBLICATION_MODEL` + `INVENTORY_MODE` resolved and payload matches; stable internal `variant_id`/SKU mapped (not replaced); PK from metadata; `SELLER_SKU`; per-UP cap / legacy limit respected | `variations-and-user-products.md` |
| `CONSISTENCY` | No contradictions across the chain below | this file |
| `RETURN_PREVENTION` | Checklist clean; mandatory question answered "no" | `return-prevention.md` |
| `COMPLIANCE` | Prohibited/restricted/regulated resolved; claims evidence-backed; brand/IP safe; no contact/external-diversion content | `compliance.md` |

## 9. Attribute semantics — required vs quality (OFFICIAL — verified 2026-08-27, search-indexed)

Do **not** merge these:

| Concept | Source | Missing → |
|---|---|---|
| `required` | `GET /categories/$CATEGORY_ID/attributes` | hard publication requirement — `PUBLICATION_STATUS = FAIL` after category resolution (ML returns an error on create). |
| `conditional_required` | `POST /categories/$CATEGORY_ID/attributes/conditional` | check pending → `PUBLICATION_STATUS = REVIEW`; check confirms required + missing → `FAIL`. |
| `new_required` | `.../attributes` tag | required under the applicable new-item condition — context-dependent, not a universal blocker. |
| `catalog_listing_required` | attribute tag / catalog context | required for the applicable catalog operation — context-dependent. |
| **technical-spec completeness** | `GET /categories/$CATEGORY_ID/technical_specs/input` | mandatory for **completeness/exposure**, **not** to publish. Missing → items get the `incomplete_technical_specs` tag and a search-ranking penalty → **`QUALITY_STATUS = REVIEW`**, `PUBLICATION_STATUS` may still `PASS`. |
| `SELLER_PACKAGE_HEIGHT/LENGTH/WIDTH/WEIGHT` | shipping mode (ME2), **not** always in `.../attributes` | can be mandatory by logistics context; missing in a resolved ME2 context → `PUBLICATION_STATUS = FAIL` (ML moderates / blocks). Do not universalise to every seller/category. |

## 10. `/items/validate` — a pre-publication check, not a guarantee

`POST /items/validate` → `PUBLICATION_STATUS` input only. It is **not** a
guarantee of acceptance, a compliance certification, a moderation pre-clearance,
a quality score, or a substitute for dynamic category checks.

- Confirmed blocking **error** → `PUBLICATION_STATUS = FAIL`.
- **Warning** / recommendation → `REVIEW` or `QUALITY_STATUS = REVIEW` (unless the
  specific warning is itself a confirmed hard condition).
- Not executed → **do not FAIL**. Whether it is mandatory in the workflow is
  INTERNAL.

## 11. `/performance` — external quality signal (post-publication)

`GET /item/$ITEM_ID/performance` (this **replaces the deprecated `/health`** for
general listing quality). It reports a score / level (`level_wording`) and
entities with `mode` = `OPPORTUNITY` or `WARNING`.

- It feeds `QUALITY_STATUS` and the optimisation queue. It is **not** Product
  Factory's readiness status, proof product facts are true, or a substitute for
  publication validation / compliance.
- Do **not** require `/performance` before the first publication — pre-publication
  `QUALITY_STATUS` is computed from the audit + known ML recommendations +
  content/image/attribute completeness. After publication it enriches findings.
- Do not state `score < X = publication invalid` unless ML documents it. A
  Product Factory auto-publish quality threshold, if any, is **INTERNAL**.
- A `/performance` "add attribute X" recommendation does not authorise inventing
  X's value — evidence still required (`compliance.md` §9).

## 12. Pre- vs post-publication audit

- **PRE_PUBLICATION** — payload readiness, compliance, quality, execution context.
- **POST_PUBLICATION** — additionally: `/moderations/last_moderation` +
  `/infractions` (preserve `reason` + `remedy`; classify affected dimension;
  resolve evidence; prepare a correction — never recreate the listing as a
  workaround), current item status, `/performance`.
- "No moderation found" is **not** proof a new payload is compliant.

## Category & product-identifier checks

- **Category** (`categories.md` §1): a `category_validated` category exists — leaf
  (`children_categories` empty), `settings.listing_allowed = true`, domain +
  attribute set fit the product. Discovery mechanism recorded but not the pass
  bar. Not yet validated because ML data is pending → REVIEW (`PUBLICATION`);
  validation executed and category unusable → FAIL. Category unusable does **not**
  by itself FAIL `CONTENT_STATUS`.
- **Product identifier** (`attributes.md`): state `KNOWN` (provenance-backed),
  `LEGITIMATELY_ABSENT` (`EMPTY_GTIN_REASON` + category `value_id`), or
  `CONDITIONAL_PENDING` (→ REVIEW). `REQUIRED_MISSING`, an invented code, or a
  competitor code with no same-product evidence → BLOCKER (`PUBLICATION`; also
  `CONTENT` if the copy asserts it). Format validity alone is not identity proof.
- **Conflicts** on category / domain / identifier (`CONFLICTING`) → human review,
  never auto-resolved; a conflicting identifier is never published.

## Title & family_name checks (`FAMILY_NAME_TITLE`)

See `titles-and-family-name.md`.

- **BLOCKER** (`PUBLICATION`; `CONTENT` if it distorts identity) — an invented
  brand / model / MPN; the title/`family_name` contradicts ProductMaster or the
  structured attributes; a manual `title` in a resolved generate-title flow; a
  required generated-title / `family_name` mechanism violated; `family_name`
  encoding the picker (`CHILD_PK`) value.
- **REVIEW** — title mode unresolved; the family/title relationship needs dynamic
  confirmation.
- **WARNING / RECOMMENDATION** — unclear/redundant wording; weak term order; a
  keyword opportunity; filler. An INTERNAL optimisation is never a BLOCKER unless
  it protects product truth or a confirmed marketplace requirement.

## Image & Product Identity checks (`IMAGES`)

See `images.md` (A OFFICIAL / B INTERNAL gallery / C Product Identity Guard).

- **BLOCKER** (`PUBLICATION`; `CONTENT`/`EXECUTION` where the asset is used) — an
  image breaks a confirmed hard ML rule (unsupported format; below the accepted
  minimum; over the size cap; over the resolved category max or below a category
  minimum; a category cover-photo moderation rule — logo/watermark/promo overlay,
  wrong background for that category); or Product Identity Guard `IDENTITY_FAIL`
  (geometry/colour/material/component/condition altered; a fabricated feature or
  accessory shown as included; a generated variant that is not the actual
  variant; an image contradicting ProductMaster).
- **REVIEW** — `IDENTITY_REVIEW` (insufficient source evidence for a generated
  detail, uncertain colour/scale, a reconstructed angle exposing unseen areas, a
  lifestyle composition implying an unsupported inclusion); a required-image /
  minimum-count rule still pending.
- **WARNING / RECOMMENDATION** (`QUALITY` only) — a recommended-only spec missed
  (1200 × 1200, ~95 %, RGB); weak crop; a redundant gallery image; a poor
  sequence; a missing optional detail shot; an internal gallery-target miss
  (7 vs ~10, no lifestyle, no infographic). Not a publication FAIL unless a
  category minimum or a critical comprehension gap makes it blocking. Marketplace
  moderation stays authoritative regardless of a local pass.

## Publication-model & inventory-mode checks

Procedural. See `variations-and-user-products.md`.

- **`PUBLICATION_MODEL`** — `PASS`: model resolved (`LEGACY` / `USER_PRODUCT`) and
  the payload strategy matches. `REVIEW` (`PUBLICATION`/`EXECUTION`): seller/item
  model pending. `FAIL`: payload definitively uses the incompatible model
  (`variations[]` for a resolved `user_product_seller`; a manual `title` where the
  new model generates it).
- **`INVENTORY_MODE`** — separate axis (`EXECUTION`). States: `STANDARD` (no
  `warehouse_management` — `PUT /items` `available_quantity`, incl. User Products
  without Multi Origem) / `MULTI_ORIGIN_SINGLE_WAREHOUSE` (`warehouse_management`,
  no `multiwarehouse`) / `MULTI_ORIGIN_MULTIWAREHOUSE` (both tags) / `UNRESOLVED`.
  `PASS`: mode resolved and the stock write matches. `REVIEW`: tags not read.
  `FAIL`: an incompatible write — `available_quantity` on `/items` for a resolved
  `MULTI_ORIGIN_*` mode; a `MULTI_ORIGIN_SINGLE_WAREHOUSE` seller spanning
  multiple warehouse/network nodes; an MLB `selling_address` stock write instead
  of `seller_warehouse`; stock to a non-seller location; an invented `store_id` /
  `network_node_id`. `available_quantity` on a non-Multi-Origem User Product is
  **not** a FAIL.
- **Identity mapping** — internal `variant_id` / SKU stable and distinct from
  every ML id; each returned ML id persisted; `family_id` not immutable business
  identity; `catalog_product_id ≠ user_product_id`.
- **Family / PK** — PARENT_PK values compatible across a forced family (conflict →
  BLOCKER); required CHILD_PK values evidence-backed (missing after an executed
  check → BLOCKER; pending → REVIEW); PK roles from domain metadata, not hardcoded.

## Compliance checks (`COMPLIANCE`)

See `compliance.md`. Feed findings to CONTENT / PUBLICATION / EXECUTION.

- Prohibited/restricted/regulated resolution run (or explicitly not needed by
  risk/context); `prohibited` → PUBLICATION + EXECUTION FAIL; `restricted` unmet
  → PUBLICATION FAIL; applicability `unresolved` for a reasonably-regulated
  product → REVIEW.
- Every material claim traces to CONFIRMED evidence; an unsupported removable
  claim is dropped (CONTENT may stay PASS); an unsupported essential claim →
  REVIEW/FAIL by evidence.
- No genuine/authentic/licensed claim without evidence; no brand used for traffic;
  compatibility ≠ affiliation. An open BPP complaint → high-priority compliance
  REVIEW; a confirmed brand-protection action → EXECUTION impact.
- No phone / WhatsApp / e-mail / external URL / off-marketplace payment / social
  handle / QR in listing content → confirmed violation = CONTENT FAIL until fixed.
- Competitor behaviour and buyer reviews are **not** compliance evidence.

## Mandatory cross-consistency check (`CONSISTENCY`)

```
ProductMaster ⇄ Category ⇄ Catalog ⇄ family_name/Title ⇄ Attributes ⇄ Description ⇄ Images ⇄ Variants
```

- Any mismatch of a material fact (material, model, quantity, colour, inclusion)
  → **BLOCKER — PRODUCT DATA CONFLICT** (`affects: [CONTENT, PUBLICATION]`).
  Example: ProductMaster `Material = Acetato`, description says `TR90`.
- Cosmetic wording differences that don't change meaning → WARNING or none.

## Per-finding shape

```json
{
  "code": "ML_REQUIRED_ATTRIBUTE_MISSING",
  "dimension": "ATTRIBUTES",
  "severity": "BLOCKER",
  "affects": ["PUBLICATION"],
  "issue": "Required attribute GTIN missing",
  "evidence": "resolved category requirement",
  "fix": "provide a verified GTIN, or resolve EMPTY_GTIN_REASON if the category supports it",
  "rule_tag": "OFFICIAL",
  "source": "references/attributes.md"
}
```

Every **BLOCKER** should give: what failed, affected dimension(s), evidence/
source, and a remedy / next action — not just "invalid listing". Every
**REVIEW** should say what resolves it (e.g. "execute the `POST` conditional-
attributes check", "obtain supplier/manufacturer evidence").

## Staleness

Dynamic checks go stale (category requirements, seller tags, inventory mode,
policy, stock location, moderation state, `/performance`). Record a conceptual
`checked_at` per check; re-check the volatile ones immediately before
publication/execution. No universal TTLs unless clearly labelled INTERNAL.

## Output

Emit the `SKILL.md` §9 JSON plus the listing draft (model, `category_id`,
`family_name`/title, attributes map, description `plain_text`, variant table,
image plan). Populate `content_status` / `publication_status` /
`execution_status` / `quality_status`, the finding arrays, `dynamic_checks_required`,
`compliance_findings`, `missing_information`, `sources_used`, and the derived
compatibility `status`. Then stop at **READY FOR REVIEW**.

## Sources

- Qualidade das publicações / listing `/performance` — https://developers.mercadolivre.com.br/pt_br/qualidade-das-publicacoes — Developers — verified 2026-08-27 (search-indexed; live 403) — `/health` discontinued → `GET /item/$ITEM_ID/performance`; `level_wording`, entity `mode` OPPORTUNITY / WARNING.
- Technical specs input / `incomplete_technical_specs` — https://developers.mercadolivre.com.br/en_us/attributes , `GET /categories/$CATEGORY_ID/technical_specs/input` — Developers — verified 2026-08-27 (search-indexed; live 403) — `required` in `.../attributes` blocks publication; technical-spec attributes affect ranking only (tag `incomplete_technical_specs`), not publication.
- Manage moderations / `/moderations/last_moderation` / `/infractions` — https://developers.mercadolivre.com.br/en_us/manage-moderations — Developers — verified 2026-08-27 (search-indexed; live 403) — `reason` always, `remedy` only when recoverable; statuses `active` / `paused` / `inactive`; preventive pauses.
- Catalog required listings / `catalog_only_restricted` — https://developers.mercadolivre.com.br/pt_br/publicacoes-necessarias-do-catalogo — Developers — verified 2026-08-27 (search-indexed; live 403) — catalog-exclusive → `under_review` + `catalog_only_restricted`; catalog-required → `opt_obey`; applies to MLB.
- SELLER_PACKAGE_* / ME2 dimensions — https://developers.mercadolivre.com.br/en_us/shipment-handling — Developers — verified 2026-08-27 (search-indexed; live 403) — package dimensions mandatory for ME2 in some categories, not always in category attributes; missing → moderated / not published.
- Validador de publicações (`POST /items/validate`) — https://developers.mercadolivre.com.br/pt_br/validador-de-publicacoes — Developers — verified 2026-08-27 (search-indexed; live 403) — pre-publication technical check; 204 clean / 400 `cause[]`; not a publication or compliance guarantee.
- Validações — https://developers.mercadolivre.com.br/pt_br/validacoes — Developers — ⚠ verify — consulted 2026-08-27 — validation flow, error vs correction request.
- Compliance / moderation / brand-protection policy sources: `references/compliance.md` §Sources.
- All other checks derive from the linked reference files.
