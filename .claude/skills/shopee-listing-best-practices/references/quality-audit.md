# Quality audit & readiness (PROVISIONAL — mirrors the Product Factory contract)

last_reviewed: 2026-08-28
phase_02_2_reviewed: 2026-08-30

Final gate. Produces the structured output in `SKILL.md` §11. No publishing.
The readiness model is `SHARED_CORE_CANDIDATE`, adopted here as a **second
marketplace implementation**, not extracted into a shared core.

## 1. Four independent readiness dimensions

| Dimension | Question | States | Driven by |
|---|---|---|---|
| **`CONTENT_STATUS`** | Can Product Factory truthfully represent the real product without inventing material facts? | `PASS` / `REVIEW` / `FAIL` | `ProductMaster` + evidence |
| **`PUBLICATION_STATUS`** | Does the assembled listing meet the currently **resolved** Shopee requirements? | `PASS` / `REVIEW` / `FAIL` | resolved category / attribute / brand / identifier / title / image / variation / price / logistics requirements; assembled-payload policy compliance |
| **`EXECUTION_STATUS`** | Can the target Shopee operation run now for this shop / API / context? Evaluated **per operation**. | `PASS` / `REVIEW` / `FAIL` | that operation's prerequisites — shop auth + scope, API availability (gap G2), inventory model, the external ids it consumes, `item_status` |
| **`QUALITY_STATUS`** | Beyond bare validity, how complete and optimised is the listing? | **`PASS` / `REVIEW` only** | recommended attributes, image completeness, SEO, description, penalty-point exposure |

A product may legitimately be `CONTENT_STATUS = PASS` with the other three
`REVIEW` (facts solid; Shopee / shop / quality context unresolved). That is
**not** a global FAIL.

### State meanings

- **`PASS`** — sufficient verified information, no known blocker.
- **`REVIEW`** — an unresolved dynamic check, missing non-blocking evidence, a
  human-judgment need, or an optimisation opportunity. Recoverable.
- **`FAIL`** — a **confirmed** condition makes the operation unsafe or invalid.
  Requires evidence. **Never FAIL merely because something has not been
  checked**: dynamic check pending → `REVIEW`; executed and confirms a mandatory
  incompatibility → `FAIL`. A non-empty `dynamic_checks_required` is **never** an
  automatic `FAIL`.
- **`QUALITY_STATUS` has no `FAIL`** — a defect severe enough to make the listing
  misleading, unsafe or wrong routes to `CONTENT` / `PUBLICATION`; an
  operation-blocking marketplace state routes to `EXECUTION`.

### Provisional-scaffold caveat

Many Shopee rules here are `SEARCH_INDEXED` / `UNVERIFIED` (`⚠ verify`). A rule
being unresolved **does not** make every listing `REVIEW`. Trigger a check only
when it is **relevant** to the product at hand. Ordinary products are not held at
`REVIEW` because a theoretical policy is unverified.

## 2. Findings — severity vs status

| Severity | Meaning | Effect |
|---|---|---|
| **BLOCKER** | a confirmed condition forcing its `affects[]` dimension(s) to `FAIL` | those dimension(s) = `FAIL` |
| **MAJOR** | a material issue needing review — not a confirmed hard breach | its `affects[]` dimension(s) = `REVIEW` |
| **WARNING** | real but non-blocking (recommended attribute missing, weak framing, SEO opportunity) | normally `QUALITY_STATUS = REVIEW`; never forces CONTENT / PUBLICATION / EXECUTION `FAIL` |
| **RECOMMENDATION** | improvement idea | no status effect |

Every finding also carries `affects: [ … ]`, a `rule_tag`
(`OFFICIAL|DYNAMIC|INTERNAL|EXPERIMENTAL|LEARNED|UNVERIFIED`) and a
`verification` (`LIVE|SEARCH_INDEXED|UNVERIFIED`). A finding may be
cross-dimension (e.g. a confirmed counterfeit → `PUBLICATION` + `EXECUTION` +
`CONTENT`).

## 3. Aggregation (deterministic, per dimension)

```
for each dimension:
  any applicable BLOCKER / confirmed FAIL   → FAIL   (QUALITY_STATUS can't reach FAIL)
  else any applicable MAJOR / REVIEW        → REVIEW
  else                                      → PASS
```

- CONTENT / PUBLICATION / EXECUTION — warnings alone do **not** prevent `PASS`.
- QUALITY — a **material** warning / optimisation finding → `REVIEW`; minor /
  RECOMMENDATION → no effect.

### Derived compatibility `status` (not the source of truth)

```
status = FAIL     if CONTENT_STATUS = FAIL or PUBLICATION_STATUS = FAIL or EXECUTION_STATUS = FAIL
status = REVIEW   else if any of the four dimensions (including QUALITY_STATUS) = REVIEW
status = PASS     otherwise
```

Label it *derived compatibility status* wherever it appears; new logic must not
depend on it.

## 4. Gates

| Gate | Requires |
|---|---|
| **Content generation** | `CONTENT_STATUS = PASS` (others may be `REVIEW`) |
| **Publication** | `CONTENT_STATUS` + `PUBLICATION_STATUS` + `EXECUTION_STATUS` all `PASS`; `QUALITY_STATUS = REVIEW` may still be acceptable |
| **Auto-publish** | the publication gate + no unresolved high-risk compliance `REVIEW` + the Product Factory approval policy. **INTERNAL** — not a Shopee rule. Human approval can resolve `REVIEW`; it can never turn a confirmed prohibition into "allowed". |

This Skill stops at **READY FOR REVIEW** regardless — it never publishes.

## 5. FAIL triggers per dimension (all require a **confirmed** condition)

| Dimension | Representative confirmed-FAIL conditions |
|---|---|
| **`CONTENT_STATUS`** | core identity / function unknown; conflicting evidence on what is sold; a model's variant identity unestablished; content needs a fabricated material fact; content materially represents a *different* product; `IDENTITY_FAIL` with no truthful option. A **removable** policy string is **not** a CONTENT failure. |
| **`PUBLICATION_STATUS`** | resolved category is not a leaf / unusable; a resolved mandatory attribute missing; brand attribute resolved-required and unset; resolved identifier required and absent (no invented code); 0 compliant 1:1 images or an image over/under a **confirmed** limit; missing weight or no enabled logistics channel in a resolved context; title / description over a **resolved** limit; price outside a **resolved** `price_limit`; `days_to_ship` outside a **resolved** range; `condition = USED` where the resolved category disallows it; confirmed prohibited product; confirmed-applicable regulated requirement with missing evidence; assembled payload still carrying contact / external-diversion content at publish. |
| **`EXECUTION_STATUS`** | invalid / absent shop auth, token or scope for the target op; rate-limited; `item_status = BANNED` / `DELETED` for an update op; a model-scoped op with no `model_id`; a stock write in a shape BR is confirmed to reject; attempted publish of a confirmed prohibited product. **Never**: requiring an `item_id` / `model_id` that the create op itself returns; failing solely because Open Platform BR access is unconfirmed (→ `REVIEW`, fall back to manual publish). |
| **`QUALITY_STATUS`** | *(no FAIL state)* — recommended attributes unfilled, no 3:4 images, thin description, weak SEO title, penalty-point exposure noted → `REVIEW`. |

## 6. Quality sub-scores (0–100) — feed mainly `QUALITY_STATUS`

`PRODUCT_ACCURACY`, `CATEGORY`, `ATTRIBUTES`, `BRAND_IDENTIFIERS`, `TITLE_SEO`,
`DESCRIPTION`, `IMAGES`, `VARIATIONS`, `PRICING`, `INVENTORY_LOGISTICS`,
`CONSISTENCY`, `RETURN_PREVENTION`, `COMPLIANCE`.

| Sub-score | Pass bar | See |
|---|---|---|
| `PRODUCT_ACCURACY` | no invented facts; values `CONFIRMED` / vetted-`INFERRED` | SKILL.md §5 |
| `CATEGORY` | resolved **and** validated (leaf, fit) | `categories.md` |
| `ATTRIBUTES` | mandatory filled with real values; recommended noted separately | `attributes.md` |
| `BRAND_IDENTIFIERS` | brand real+evidence-backed or "Sem marca"; identifier state valid, never invented | `brand-and-identifiers.md` |
| `TITLE_SEO` | within resolved length; Title Case; no prohibited content; specs only where legitimate; no ranking folklore scored as fact | `titles-and-seo.md` |
| `DESCRIPTION` | within resolved limit; no banned content; claims backed | `descriptions.md` |
| `IMAGES` | confirmed hard specs met; count within resolved limits; cover rules; recommended-only specs are WARNINGs; every asset evidence-backed & `IDENTITY_PASS`; model-correct | `images.md` |
| `VARIATIONS` | tiers / options / models within resolved caps; stable internal `variant_id` / SKU mapped (not replaced); each model's identity established | `variations.md` |
| `PRICING` | present and within resolved `price_limit`; no unsupported reference price | `pricing.md` |
| `INVENTORY_LOGISTICS` | stock level correct; weight / dimensions / channel / DTS resolved and within limits | `inventory.md`, `logistics.md` |
| `CONSISTENCY` | no contradictions across the chain below | this file |
| `RETURN_PREVENTION` | checklist clean; mandatory question answered "no" | `return-prevention.md` |
| `COMPLIANCE` | prohibited / restricted / regulated resolved; claims backed; brand / IP safe; no contact / diversion content | `compliance.md` |

## 7. Mandatory cross-consistency check (`CONSISTENCY`)

```
ProductMaster ⇄ Category ⇄ Title ⇄ Attributes ⇄ Brand/Identifiers ⇄ Description ⇄ Images ⇄ Tier Variation/Models
```

Any mismatch of a material fact (material, model, quantity, colour, inclusion,
which model an image or option name denotes) → **BLOCKER — PRODUCT DATA CONFLICT**
(`affects: [CONTENT, PUBLICATION]`). Cosmetic wording differences → WARNING or
none.

## 8. Pre- vs post-publication audit

- **PRE_PUBLICATION** — payload readiness, compliance, quality, execution context.
- **POST_PUBLICATION** — additionally: current `item_status`, any moderation /
  penalty-point state (`moderation-and-enforcement.md`) — preserve `reason` +
  `remedy`; classify `affects[]`; never recreate the listing as a workaround.
  "No moderation found" is **not** proof a payload is compliant.
- **`/performance` analogue (Phase 02.2):** community SDKs expose a
  **content-diagnosis** API — `product/get_item_content_diagnosis_result`
  (`item_id`) and `product/get_item_list_by_content_diagnosis` — the closest
  Shopee equivalent to Mercado Livre's `/performance`. It is `SEARCH_INDEXED`
  (schema + BR availability `UNVERIFIED`), it operates **post-creation** (not a
  pre-publish gate), and it feeds `QUALITY_STATUS` only — never a
  publication/execution FAIL. Optional check `resolve_content_diagnosis`
  (`api-and-auth.md` §5). Until it is verified for BR, `QUALITY_STATUS` is driven
  by our own checks + documented Shopee recommendations + (post-publication)
  penalty exposure.

## 9. Dynamic-check staleness

Category requirements, attribute sets, brand list, limits, inventory model,
policy state, `item_status` and penalty state all go stale. Record a conceptual
`checked_at` per check; re-check the volatile ones immediately before any
publish / execution attempt. No universal TTLs unless labelled INTERNAL.

## 10. Output

Emit the `SKILL.md` §11 JSON plus the listing draft (Item + `category_id` +
title + attributes map + brand + identifier state + description + model/variant
table + image plan with per-image role and model mapping). Populate
`audit_mode`, the four statuses (with `execution_operation`), the finding arrays,
`compliance_findings`, `missing_information`, `dynamic_checks_required`,
`sources_used`, and the derived compatibility `status`. Then stop at
**READY FOR REVIEW**.

Any change to the dimension list, status states, severity scale, aggregation rule
or finding shape must be mirrored in `SKILL.md` §11 (and, once stable, repo
`CLAUDE.md`).

## Sources

- Readiness model, aggregation, gates, finding shape, cross-consistency —
  `.claude/skills/mercado-livre-listing-best-practices/references/quality-audit.md`
  and `SKILL.md` §7–§9 — internal — architectural reference only; adapted here as
  Shopee-worded rules for a second marketplace.
- Shopee-specific FAIL triggers, "no `/performance` API", operation list —
  `research/shopee-listing-skill/discovery-report.md` §25, §22, §26.
