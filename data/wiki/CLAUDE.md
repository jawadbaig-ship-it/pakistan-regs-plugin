# CLAUDE.md — Pakistan Lender Regulatory Wiki Schema

This file tells the LLM how to maintain this wiki. Read it at the start of every session.

## Purpose

A persistent, compounding knowledge base of Pakistan's lender regulatory landscape — issued by SBP, SECP, FMU, MoITT, PTA, NADRA and other regulators — organised so that an agentic compliance pipeline can validate PRDs, deployments and customer journeys against statutory and prudential requirements, and a chat agent can answer "is X allowed?" with citations down to circular-section level.

## Three-layer architecture

```
raw/             Immutable source PDFs/HTML — never edited, only added to
wiki/            LLM-maintained markdown (this directory)
chunks.jsonl     Embeddings-ready chunked text (regenerated, not edited)
graph.json       Knowledge graph (regenerated from wiki frontmatter + body)
```

## Wiki directory layout

```
wiki/
├── CLAUDE.md                  This schema (read first)
├── index.md                   Content-oriented catalog (sources / entities / concepts)
├── log.md                     Append-only chronological log
├── <regulation>.md            One per regulation/circular/act (115+ files)
├── concepts/<concept>.md      Cross-cutting concepts (KFS, eCIB, CDD, etc.)
└── entities/<entity>.md       Regulators (SBP, SECP, FMU…) and entity types (Bank, MFB, NBFC…)
```

## Page types and required frontmatter

### Regulation pages (one per circular/act/framework)
```yaml
---
doc_id: SBP_eCIB_Master              # stable kebab/snake id
name: "SBP eCIB Master Circular"
type: regulation
regulator: SBP
reference: "BC&CPD Circular No. 06 of 2021"
date: "2021-06"
applicable_to: "Banks, DFIs, MFBs, Digital Banks"
source_url: https://...
status: active                       # active | superseded | draft
supersedes: []                       # list of doc_ids this replaces
superseded_by: []                    # list of doc_ids that replace this
tags: [sbp, ecib, credit-reporting]
---
```

Body sections (in order):
1. One-line tagline (italic, under H1)
2. `## Purpose & Scope`
3. `## Key Sections / Requirements`
4. `## Auto-Extracted Section Headings` (from source PDF)
5. `## Detected Keywords`
6. `## Related concepts` — bulleted `[[concepts/...]]` wikilinks
7. `## Related entities` — bulleted `[[entities/...]]` wikilinks
8. `## See also` — related regulations as `[[doc_id]]`

### Concept pages (cross-cutting topics)
```yaml
---
name: "Key Fact Statement (KFS)"
type: concept
aliases: ["KFS", "Key Fact Statement"]
tags: [consumer-protection, disclosure]
---
```
Body: definition · why it matters · governing regulations (with `[[doc_id]]` wikilinks) · key obligations · common pitfalls.

### Entity pages (regulators and regulated-entity types)
```yaml
---
name: "Microfinance Bank (MFB)"
type: entity
entity_kind: regulated_entity        # regulator | regulated_entity | adjacent_authority
regulator: SBP
tags: [mfb, microfinance]
---
```
Body: definition · enabling statute · primary regulator · key prudential rules · linked regulations.

## Operations

### Ingest a new source
1. Place the PDF/HTML under `raw/` with a clear filename.
2. Read the source. Note the regulator, date, reference number, applicable entities.
3. Create or update `wiki/<doc_id>.md` with full frontmatter and the 8 body sections.
4. Detect references to existing concepts/entities — add `[[concepts/...]]` and `[[entities/...]]` wikilinks.
5. Update each linked concept/entity page: add a backlink under "Appears in".
6. If this regulation supersedes an older one, set `supersedes` and update the older doc's `superseded_by` and `status: superseded`.
7. Append to `wiki/log.md`: `## [YYYY-MM-DD] ingest | <doc_id> | <one-line summary>`.
8. Re-run `python tools/build_graph.py` to refresh `graph.json` and `chunks.jsonl`.

### Answer a query
1. Read `index.md` to identify candidate regulations / concepts / entities.
2. Drill into 2–5 relevant pages.
3. Synthesise an answer with `[[doc_id]]` and section-level citations (e.g. "BPRD C13/2008 §2 — Fair Debt Collection").
4. If the question revealed a new comparison or analysis worth keeping, file it as a new wiki page (e.g. `wiki/topics/<question-slug>.md`) and append a `query` entry in `log.md`.

### Lint
- Find regulations whose `applicable_to` mentions an entity but that lack a wikilink to it.
- Find concepts whose definition is `TODO` or empty.
- Find regulations marked `superseded` that are still cited by `active` ones.
- Find orphan pages (no inbound links anywhere in the wiki).
- Find aliases collisions (two concepts claiming the same alias).

## Citation style

When generating answers for downstream consumers (PRD validators, chat agent), always cite as:

> *"Pre-disbursement KFS is mandatory for digital nano-loans. Source: SECP Circular 10/2023 (digital nano-lending requirements)."* — `[[SECP_10_2023]]` · [original PDF](source_url) · §<n>

The compliance pipeline expects `doc_id` in the citation so it can resolve to a chunk in `chunks.jsonl`.

## Tag taxonomy (controlled vocabulary)

Domains: `consumer-protection`, `aml-cft`, `credit-reporting`, `customer-onboarding`, `digital-lending`, `digital-banking`, `payment-systems`, `outsourcing`, `cloud`, `tech-risk`, `cyber`, `data-protection`, `prudential`, `capital`, `liquidity`, `recovery`, `disclosure`, `governance`, `microfinance`, `sme`, `agriculture`, `housing`, `islamic`, `green`, `kyc`, `cdd`, `edd`, `biometric`, `ecib`, `classification`, `collection`, `provisioning`, `sanction`, `exposure`, `tenor`, `consent`, `data-privacy`, `wallet`, `markup`, `ifrs`, `shariah`, `agent`, `identity`, `tier`, `pilot`, `sandbox`, `otp`, `tat`, `kfs`, `key-fact`, `dbr`, `ctr`, `str`, `beneficial-owner`, `paid-up-capital`, `pilot-phase`, `risk-based`, `whitelist`, `gop-fmu`, `gop-mof`, `gop-moitt`, `gop-sbp`, `sbp-secp`, `iba-industry`, `pide-research`, `tasdeeq-credit-bureau`, `industry-news`, `legal-3rd-party`, `bank-guidance-3rd-party`, `scribd-3rd-party-copy`.

Add to this list rather than inventing parallel tags.

## Authoring rules

- **No prose-only paragraphs in regulation pages.** Use the 8-section template — chat answers and the graph builder rely on it.
- **Every regulation must declare `applicable_to`.** This drives the compliance matrix.
- **Wikilinks use `[[doc_id]]` for regulations**, `[[concepts/<slug>]]` for concepts, `[[entities/<slug>]]` for entities. Do not use bare slugs.
- **Status field is mandatory.** `active` unless explicitly superseded. Update `supersedes` / `superseded_by` symmetrically.
- **Keep summaries terse but specific.** "Imposes APR cap" not "imposes various caps". Numbers, section refs, entity names belong in the body.
- **Source links are immutable.** If a URL breaks, mark `status: stale-link` and add a note — do not silently delete.

## When the user is wrong

If the user asks for something that conflicts with a regulation in the wiki, surface the conflict rather than complying silently. Quote the specific clause and `[[doc_id]]`.
