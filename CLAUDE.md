# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

There is no application code, build system, or test suite here. The repository
contains a single **Claude Agent Skill**:
`skills/mercado-livre-listing-best-practices/`. Work in this repo means authoring
and maintaining that Skill's Markdown.

The Skill turns a structured `ProductMaster` into a Mercado Livre **Brasil**
listing draft plus a structured quality-audit JSON. It respects Mercado Livre's
current object model (Item / User Product / Family / Catalog). It **never
publishes** — it stops at `READY FOR REVIEW`.

## Skill layout and how it is meant to be read

- `SKILL.md` — the entry point and the only file loaded up front. §4 is a
  dispatch table ("Question you are answering → Read `references/X.md`"); §6 is
  the 16-step creation workflow; §9 is the exact output JSON shape.
- `references/*.md` — 15 self-contained topic files (categories, attributes,
  images, descriptions, variations-and-user-products, catalog, titles-and-family-name,
  pricing-and-commercial, return-prevention, quality-audit, etc.). These are read
  **on demand**, one at a time, driven by `SKILL.md` §4 — never all at once.

When you add or rename a reference file, update the `SKILL.md` §4 map and any
cross-links in sibling reference files in the same change.

## Conventions that must be preserved when editing

- **Rule classification tags.** Every recommendation the Skill emits carries
  exactly one tag: `OFFICIAL` (in Mercado Livre docs — cite it), `DYNAMIC`
  (must be fetched from the ML API at listing time — never hardcode), `INTERNAL`
  (our own good practice), `EXPERIMENTAL` (unproven hypothesis), `LEARNED` (from
  our historical performance data). Never let INTERNAL/EXPERIMENTAL/LEARNED be
  phrased as an official ML rule. Official docs beat external sources; flag
  conflicts for human review.
- **DYNAMIC values are never written as constants** anywhere in the Skill:
  `category_id`, attribute ids/value lists/tags, `max_pictures_per_item(_var)`,
  `max_title_length`, listing types, the per-User-Product condition cap, catalog
  matches. Reference them by API endpoint (`GET /categories/$ID/attributes`,
  `domain_discovery/search`, `POST /items/validate`, …), not by value.
- **`⚠ verify`** marks any rule written without a successful live fetch of the
  official page (developers.mercadolivre.com.br returns 403 to bots). Keep the
  marker until a maintainer re-reads the live doc, then remove it.
- **Reference-file metadata.** Each `references/*.md` opens with `last_reviewed:`
  (and, where relevant, `source_last_updated:` / `volatile: true`) and **ends
  with a `## Sources` block**: title, URL, origin (Developers / Central de
  Vendedores / Central de Ajuda / external), update date if known, consultation
  date, and which rules were derived from it. Summarize sources — never paste
  large excerpts of ML documentation.
- **Freshness.** Bump `last_reviewed` (and `SKILL.md` `last_reviewed`) whenever
  you re-verify a file. Re-verify the volatile files (`product-structure`,
  `variations-and-user-products`, `categories`, `attributes`, `images`,
  `catalog`, `pricing-and-commercial`) at least quarterly.
- **Accuracy over conversion.** When listing appeal and product faithfulness
  conflict, faithfulness wins — this framing runs through every file.

## Audit / output contract

`SKILL.md` §8–§9 and `references/quality-audit.md` define the output. It carries
**four independent readiness dimensions** — `content_status`,
`publication_status`, `execution_status` (each `PASS` / `REVIEW` / `FAIL`) and
`quality_status` (`PASS` / `REVIEW` only) — plus a **derived compatibility**
`status` (not the source of truth: `FAIL` if any of content/publication/execution
is FAIL, else `REVIEW` if any dimension is REVIEW, else `PASS`), the 12 quality
sub-dimension scores, and finding arrays. Each finding is
`{ code, severity, affects[], rule_tag, issue, … }`; severity is `BLOCKER` /
`MAJOR` / `WARNING` / `RECOMMENDATION`. A `BLOCKER` forces its `affects[]`
dimension(s) to `FAIL`; a `MAJOR` forces them to `REVIEW`. A **pending** dynamic
check keeps the relevant dimension at `REVIEW` — a non-empty
`dynamic_checks_required` is **never** an automatic `FAIL`; only a check that has
been *executed and confirms an incompatibility* fails its dimension. Any change
to the dimension list, status states, severity scale, aggregation rule, or
finding shape must be mirrored across `SKILL.md` and `references/quality-audit.md`.
