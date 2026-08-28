# Compliance — a resolution procedure, not a frozen catalogue

last_reviewed: 2026-08-28
volatile: true
classification: procedure INTERNAL; Shopee policy references OFFICIAL (verification SEARCH_INDEXED, `⚠ verify`)

## 1. Model to build

```
product / claims / context
        ↓  Shopee policy resolution   (DYNAMIC — NOT a frozen list)
ComplianceFinding { rule, source, classification, scope, evidence, status, affects[], remedy }
        ↓
readiness impact  (CONTENT / PUBLICATION / EXECUTION — never a fifth status)
```

**Do not freeze a prohibited-products list into the Skill.** Build the resolution
*procedure*; keep any examples as dated illustrations tagged `SEARCH_INDEXED` +
`⚠ verify`.

## 2. Working states (INTERNAL until Shopee terminology is mapped)

| State | Meaning |
|---|---|
| `ALLOWED` | no restriction resolved for this product / context |
| `RESTRICTED` | listable only under conditions |
| `AUTHORIZATION_REQUIRED` | a sub-state of `RESTRICTED` — needs a Seller-Center release request before listing |
| `PROHIBITED` | cannot be listed at all |
| `UNRESOLVED` | applicability not yet determined |

Shopee BR's own **"Política de Produtos Proibidos e Restritos"** (help art.
76226; seller-edu art. 3304) uses a **prohibited vs restricted** split — use that
as the backbone once mapped.

## 3. What discovery found (all `SEARCH_INDEXED`, `⚠ verify`)

- **Proibidos (prohibited):** cannot be listed. Includes items lacking required
  homologation from **ANVISA / ANATEL / INMETRO / MAPA / ANS** (and effectively
  Exército / Polícia Federal for controlled items); counterfeits; many
  used / hygiene items; fractioned prescription meds, hormones, anaesthetics;
  cosmetic testers / samples / decants; etc. *(illustrative, dated 2026-08-28,
  `⚠ verify`)*
- **Restritos (restricted):** listable only with authorisation / conditions.
  Seller requests release via Seller Center (edu art. 12544). Examples: eyewear
  frames, food & beverages, alcohol, dairy, certain electronics; remould tyres
  (must carry "Remold" in the title + INMETRO number in the body).
  *(illustrative, `⚠ verify`)*

### Regulators (Brazil) — applicability, not assumption

| Regulator | Scope | Handling (reconstructed) |
|---|---|---|
| **ANVISA** | cosmetics, health / food, supplements, some devices | valid registration / notification required; testers / samples / fractioned Rx prohibited |
| **ANATEL** | telecom / RF devices (wi-fi, Bluetooth) | homologation required; uncertified → prohibited |
| **INMETRO** | toys, electricals, appliances, auto parts, textiles, tyres, PPE, childcare | certification required; certificate / registration number often required in the listing |
| **MAPA** | agri / veterinary / pet food / some foods | registration required; some items prohibited |
| **ANS** | health-plan-related | referenced as a homologation authority |
| Exército / PF | controlled products (airguns, chemicals, …) | heavily restricted / prohibited |

Principle (shared with the Mercado Livre Skill): **possible applicability →
resolve**; **confirmed applicable + missing requirement → `PUBLICATION_STATUS`
blocker.** Never assume applicability; never invent a registration number.

## 4. Claim safety (`SHARED_CORE_CANDIDATE`)

`claim strength ≤ evidence strength`. No invented certification, medical
efficacy, authenticity, licence, compatibility, safety or regulatory approval.

- An unsupported **removable** claim → drop it; `CONTENT_STATUS` may stay `PASS`;
  record a finding.
- An unsupported **essential** claim that cannot be dropped → `REVIEW` / `FAIL`
  by evidence.

## 5. Brand / IP

- No brand used for traffic; compatibility ≠ affiliation.
- IP-gated categories need brand authorisation
  (`references/brand-and-identifiers.md`).
- A rejected custom brand auto-reverts the listing to "Sem marca".

## 6. Contact info / external diversion (removable string)

Prohibited in titles, descriptions, images, shop decoration and package inserts:
links, phone numbers, QR codes, social handles, WhatsApp invitations, "pagar
fora", any redirection off Shopee.

Handling: **drop the string** → `CONTENT_STATUS` may stay `PASS`; record a
compliance / content finding; `PUBLICATION_STATUS = FAIL` **only if** the
assembled payload still carries it at publish time
(`resolve_contact_diversion_clean`).

## 7. Pre-publication validation gap

No dedicated Shopee pre-publication validator has been confirmed (no
`POST /items/validate` analogue found). **The negative is not proven** — say
"no dedicated validator endpoint has been verified", not "Shopee has no
validator". Consequence: `PUBLICATION_STATUS` leans on up-front limit / attribute
fetches + local payload checks; the `add_item` response is the authoritative
gate. (SKILL.md gap G5.)

## 8. Readiness impact

- Prohibited / restricted / regulated resolution run (or explicitly not needed by
  risk / context).
- `PROHIBITED` → `PUBLICATION_STATUS = FAIL` + `EXECUTION_STATUS = FAIL`
  (attempted prohibited publish).
- `RESTRICTED` / `AUTHORIZATION_REQUIRED` unmet → `PUBLICATION_STATUS = FAIL`.
- Applicability `UNRESOLVED` for a reasonably-regulated product →
  `PUBLICATION_STATUS = REVIEW` (`resolve_compliance_applicability`).
- Every material claim traces to `CONFIRMED` evidence.
- Competitor behaviour and buyer reviews are **not** compliance evidence.

## Sources

- Prohibited / restricted split; regulators ANVISA / ANATEL / INMETRO / MAPA /
  ANS; homologation-absent → prohibited — `help.shopee.com.br` art. 76226,
  `seller.shopee.com.br/edu` art. 3304 — Central de Ajuda / Centro de Educação —
  consulted 2026-08-28 — `SEARCH_INDEXED`.
- Restricted-item release flow — `seller.shopee.com.br/edu` art. 12544 — Centro
  de Educação — consulted 2026-08-28 — `SEARCH_INDEXED`.
- Contact / external-diversion prohibition — `help.shopee.com.br`, BR educators
  — Central de Ajuda / external — consulted 2026-08-28 — `SEARCH_INDEXED`, HIGH.
- Resolution-procedure + finding-struct + removable-string model —
  `.claude/skills/mercado-livre-listing-best-practices/references/compliance.md`
  — internal — architectural reference only.
- Full detail: `research/shopee-listing-skill/discovery-report.md` §18, §19, §20.
