---
name: pakistan-regs-cite
description: Produce a formal, citation-ready reference block for a Pakistan lender regulation — circular number, page, section, verbatim quote, and deep-linked PDF URL. Use whenever the user asks "cite the rule that says X", "give me the source for X", "what's the exact circular reference for X", "I need a quote for the legal memo / PRD footnote / Confluence page", or asks to format an existing finding for inclusion in a document. Also use when the user has already received an answer from another skill and wants the source upgraded into a publishable citation block.
---

# Pakistan Regulatory Citation Generator

You produce **publication-quality citations** — the kind that go into a PRD footnote, a Confluence compliance page, a Slack thread to legal, or a paragraph in a board memo. Each output must be self-contained and traceable.

## Step 1 — Identify the target regulation

The user will provide one of:
- A `doc_id` (e.g., `SBP_eCIB_Master`)
- A topic / question (e.g., "the rule that bans contact-list access")
- A vague reference (e.g., "the SBP fair debt collection circular")

If a topic, first run a wiki search (same flow as `pakistan-regs-search`): start at `data/wiki/index.md` or `data/wiki/concepts/<slug>.md` to identify the right `doc_id`. If multiple regulations are relevant, list them all and produce a citation block per regulation.

## Step 2 — Open the regulation's wiki page and find the right chunk

1. Read `data/wiki/<slug-of-doc-id>.md` for the curated context (Purpose, Key Sections, Applicable To).
2. Open `data/chunks.jsonl` and find the chunks where `doc_id == <target>`. Each chunk has:
   - `chunk_id` (e.g., `SBP_eCIB_Master::p2::c1`)
   - `page` (PDF page number)
   - `section_anchor` (e.g., `§4.2 General Instructions`)
   - `source_url_with_page` (URL + `#page=N`)
   - `text` (the verbatim passage)
3. Pick the chunk that *actually contains* the rule the user is asking about. If you can't find a single chunk that covers it, return the closest 2-3 chunks and clearly say "spans multiple sections".

## Step 3 — Output format

Produce **two variants** so the user can pick what fits their document.

### Variant A — Inline citation (for prose)

```
The rule on <topic> is set by <doc_name> (<reference>): "<verbatim quote, ≤30 words>" ([source][1]).

[1]: <source_url_with_page>
```

### Variant B — Block citation (for footnotes / compliance reports)

```
> **Rule:** <one-line plain-English restatement>
>
> **Source:** <doc_name> · <reference> · <date>
> **Section:** <section_anchor>
> **Verbatim:** "<quote, ≤40 words>"
> **Applicable to:** <applicable_to from frontmatter>
> **Link:** <source_url_with_page> (opens at page <N>)
> **Wikilink:** [[<doc_id>]]
> **Chunk:** `<chunk_id>` (resolvable in chunks.jsonl)
> **Status:** <active | superseded | stale-link from frontmatter>
```

Always produce both variants. The user pastes whichever fits their target medium.

## Step 4 — Multi-regulation citations

If the rule spans regulations (e.g., the eCIB obligation lives in both `[[SBP_eCIB_Master]]` and `[[GoP_BCO_1962]]`), output one block per source, ordered from most-specific to most-general:

```
1. Operational rule  — [[SBP_eCIB_Master]] §4.2 (monthly reporting)
2. Statutory basis   — [[GoP_BCO_1962]] §25-A (confidentiality)
3. Enabling Act      — [[GoP_CBA_2015]] (private bureau framework)
```

## Step 5 — Quality bar

A citation block is acceptable only if all of these are true:

- [ ] `doc_id` exists in `data/wiki/`
- [ ] `chunk_id` exists in `data/chunks.jsonl` (verify by grep)
- [ ] `source_url_with_page` is from the chunk's `source_url_with_page` field, not constructed by hand
- [ ] Verbatim quote is from `text` field of a real chunk (no paraphrase, no fabrication)
- [ ] Quote is ≤ 40 words; if longer is needed, link instead of inlining
- [ ] Section anchor is from the chunk, not invented
- [ ] If the page has `status: stale-link`, note it in the citation block

**If any of these checks fail, refuse to produce the citation and tell the user why.** A fabricated citation is worse than no citation in a compliance context.

## Step 6 — Edge cases

- **Stale-link pages**: 17 of the 115 regulations have a dead `source_url` (SBP moved URLs after the source spreadsheet was compiled). Their wiki page carries the curated content but no fresh PDF. Cite the wiki page, mark "Status: stale-link", and recommend the user verify against the latest SBP / SECP listing.
- **HTML-only sources (portal pages)**: no `#page=N` deep link exists. Cite the URL as-is and use chunk index instead of page number.
- **Curated summary chunks** (`chunk_type: curated_summary`, one per doc): these are the maintainer's synthesis, not verbatim from the regulator. Use them only for high-level positioning paragraphs, never as the verbatim source of a rule.
- **Primary acts** (e.g., BCO 1962, AML Act 2010, MFI Ordinance 2001): cite by section number (e.g., `§25-A BCO 1962`). Statutes often outlive specific URLs; the section ref is what matters.

## Worked example

User: *"Cite the regulation that bans digital lenders from accessing the borrower's phone contact list."*

You produce:

> **Variant A (inline):**
>
> SECP prohibits digital lenders from harvesting a borrower's contact list or photo gallery, irrespective of user consent: *"Digital lenders are strictly barred from accessing a borrower's phone book, contact lists, or photo galleries"* ([Circular 12/2024][1]).
>
> [1]: https://www.secp.gov.pk/document/circular-no-12-of-2024-requirements-for-nbfcs-engaged-in-digital-lending/
>
> **Variant B (block):**
>
> > **Rule:** Digital lenders must not access borrower phone contacts, photo galleries or SMS — prohibition is absolute and overrides user consent.
> >
> > **Source:** SECP Circular 12/2024 — Updated Requirements for NBFCs Engaged in Digital Lending · 2024-05-16
> > **Section:** Data-privacy controls
> > **Verbatim:** "Digital lenders are strictly barred from accessing a borrower's phone book, contact lists, or photo galleries."
> > **Applicable to:** All SECP-licensed Lending NBFCs operating Digital Lending Apps; partners and outsourced service providers
> > **Link:** https://www.secp.gov.pk/document/circular-no-12-of-2024-requirements-for-nbfcs-engaged-in-digital-lending/
> > **Wikilink:** [[SECP_12_2024]]
> > **Chunk:** `SECP_12_2024::summary`
> > **Status:** active
>
> See also: `[[SECP_8_2024]]` (parent circular), `[[SECP_10_2023]]` (nano-lending caps), and `[[concepts/data-protection]]` for the wider data-privacy concept.

That's the bar. Anything less and the citation is not publishable.
