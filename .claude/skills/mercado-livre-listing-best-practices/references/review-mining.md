# Review mining

last_reviewed: 2026-08-27
classification: INTERNAL methodology

Goal: use reviews and Q&A from similar products to make our listing answer real
buyer concerns before purchase. Use only when that data is accessible.

## Sources to mine

- Reviews on equivalent products (ours, competitors', catalog page).
- Q&A (perguntas) on those listings.
- Return reasons / complaints if available from our own history (future: LEARNED).

## Extract

| Signal | Use it for |
|---|---|
| Recurring **questions** | Missing attribute or unclear description → add/clarify |
| **Objections** ("achei pequeno", "não vem a fonte") | "Antes de comprar" note + dimension/in-box image |
| **Complaints** ("cor diferente da foto", "material frágil") | Fix image accuracy; state material honestly; set expectations |
| **Praise** ("montagem fácil", "encaixe perfeito") | Legit benefit to highlight (only if true for our product) |
| **Expectations** (what buyers assumed) | Pre-empt the wrong assumption explicitly |
| **Misunderstood features** | Add a plain-language explanation |

## Convert to listing changes

For each recurring theme, decide the target surface:

- **Attribute** — if there's a structured field for it (preferred).
- **Description → "Antes de comprar"** — sizing, compatibility caveats, what's not
  included, illustrative-image disclaimer.
- **FAQ block** (in description) — the top 3–5 real questions, answered plainly.
- **Image** — dimensions shot, in-box shot, close-up of the doubted detail.
- **Video** (if used) — demonstration for assembly/operation confusion.

## Rules

- Only carry over praise/benefits that are **true for our exact product**
  (CONFIRMED evidence). Another product's strength is not automatically ours.
- Don't quote reviews verbatim into the listing.
- A complaint about a competitor's build quality is not a claim we can invert into
  a superiority statement without proof.

## Output

A `review_intel` block: top questions, top objections, top complaint themes, and
the concrete listing change proposed for each (with target surface).

## Sources

Methodology is INTERNAL. Buyer-generated content on Mercado Livre is not official
documentation and must be treated as unverified.
