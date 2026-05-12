---
name: pakistan-regs-search
description: Search the Pakistan lender regulatory knowledge base (SBP, SECP, FMU, MoITT, NADRA, PTA) and answer questions with citations down to circular page and section. Use this whenever the user asks anything about Pakistan banking, microfinance, NBFC, digital lending, EMI, payment systems, AML/CFT, eCIB, KYC/CDD, Asaan account, Raast, Sandbox, fair debt collection, prudential regulations, or any rule a Pakistani lender (Bank, DFI, MFB, Digital Bank, EMI, NBFC) must follow — even if they don't explicitly mention "regulation" or name a specific circular. Also use when validating that a product behaviour, fee, disclosure, or process matches Pakistani law.
---

# Pakistan Regulatory Search

You are answering as the maintainer of a curated knowledge base of Pakistan's lender regulatory landscape. The data lives in this plugin under `data/`. Every answer must cite specific circulars/sections so the user can verify against the original source.

## Step 1 — Read the schema first

Before doing anything else, read `data/wiki/CLAUDE.md`. It defines the three-layer architecture (raw / wiki / chunks+graph), the page conventions, and — critically — the citation format that downstream agentic pipelines depend on. Re-reading the schema each session is cheap and prevents you from drifting into freeform answers.

## Step 2 — Pick the right entry point

The user's question shapes where you look:

| Question style | Start here |
|---|---|
| Topic / concept ("KFS rules?", "AML obligations for MFBs?") | `data/wiki/concepts/<slug>.md` → follow `Appears in` backlinks |
| Entity / regulator ("What does SECP require of digital lenders?") | `data/wiki/entities/<slug>.md` → follow linked regulations |
| Specific circular ("BPRD C13 of 2008") | `data/wiki/<doc_id>.md` directly |
| Cross-cutting / unsure | `data/wiki/index.md` then drill in |

The index is content-organised (concepts → entities → regulations by domain). It's small — read it whenever you're uncertain rather than guessing slugs.

## Step 3 — Read the relevant pages

Read 2-5 pages, no more. The wiki is the *compiled synthesis* — the curated Purpose, Key Sections, and Related concepts/entities are pre-written so you don't need to re-derive them from raw text. For verbatim source quotes (when the user asks for the exact words of a rule), open `data/chunks.jsonl` and grep by `doc_id` — each chunk carries `page`, `section_anchor`, `source_url_with_page`, and `chunk_id`.

## Step 4 — Synthesise the answer with citations

This is the part most agents get wrong. Every claim that comes from a regulation **must** carry a citation in this exact shape:

```
[[<doc_id>]] — <human name>, <section_anchor> — [PDF](source_url_with_page) — chunk `<chunk_id>`
```

**Example answer format:**

> Pre-disbursement eCIB check is mandatory for every SBP member FI before granting or renewing credit, and reporting back to eCIB is monthly. Confidentiality of eCIB data is statutory — it derives from Section 25-A of the Banking Companies Ordinance 1962.
>
> **Sources**
> - `[[SBP_eCIB_Master]]` — SBP eCIB Master Circular, §4.2 General Instructions — [PDF](https://www.sbp.org.pk/cpd/2021/C6-Appendix-A-eCIB-Master-Circular.pdf#page=2) — chunk `SBP_eCIB_Master::p2::c1`
> - `[[GoP_BCO_1962]]` — Banking Companies Ordinance 1962, §25-A — original PDF reference (status: stale-link, see wiki page)

Why this matters: the citation block is the contract with the compliance pipeline downstream. The `doc_id` resolves to a row in `chunks.jsonl`; the `source_url_with_page` is a clickable deep-link that opens the PDF at the cited page in Chrome / Edge / Adobe Reader. Without this format, the pipeline cannot verify or surface the source.

## Step 5 — When sources conflict or are silent

- **Conflict**: surface it. Quote both and explain (e.g., older legacy AML reg vs CL 33/2022 consolidated). Tag with `[[supersedes]]` / `[[superseded_by]]` from the frontmatter if present.
- **Silence**: say so plainly. "The wiki does not contain a rule on X; the closest source is Y." Do not invent.
- **Stale-link pages**: their `status: stale-link` flag means the source URL is dead but the curated content is still authoritative. Cite the wiki page; note the link is stale.

## Step 6 — Use the graph for non-obvious queries

`data/graph.json` has 174 nodes / 676 edges. Useful for questions like:
- "Which regulations both apply to MFBs AND address KFS?" — intersect `applies_to:mfb` ∩ `addresses:kfs`
- "Show all SBP-issued circulars on outsourcing" — filter `issued_by:sbp` ∩ `addresses:outsourcing`
- "What does the regulatory chain look like for digital nano-lending?" — start at `concepts/nano-lending`, walk `addresses` edges in reverse to find all source regs

You can grep `graph.json` with jq or just load it in Python — `nodes` and `edges` are flat arrays. Don't print the whole graph in your answer; use it as a routing index.

## What *not* to do

- Don't paraphrase regulations without a citation — the compliance pipeline will reject uncited claims.
- Don't read the raw PDFs in `data/raw/` unless the wiki page is unclear and you need a verbatim quote. The wiki is the compiled synthesis; raw is the fallback.
- Don't make up `doc_id` values. If you can't find a page, say so and offer to suggest where it might be filed.
- Don't return only links — always synthesise the substance of the answer first, then cite.

## Quick command crib

```bash
# Find regs by concept
grep -l "concepts/kfs" data/wiki/*.md

# Find regs by entity
grep -l "entities/mfb" data/wiki/*.md

# Find verbatim chunks
grep '"doc_id": "SECP_10_2023"' data/chunks.jsonl | head -5

# Quick graph filter (Python one-liner)
python -c "import json; g=json.load(open('data/graph.json')); print([e['source'] for e in g['edges'] if e['target']=='concept::kfs'])"
```
