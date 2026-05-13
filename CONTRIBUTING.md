# Contributing to the Pakistan Regulatory Wiki

Thank you for adding to this knowledge base. The wiki is the **compiled
synthesis** of Pakistan's lender regulatory landscape; the chunks, graph
and skills are *derived* from it. Treat the wiki markdown as the source of
truth — never edit `chunks.jsonl` or `graph.json` by hand.

## TL;DR

```bash
# 1. Edit a wiki page (or add a new regulation)
vim wiki/<doc_id>.md

# 2. Verify
py -3 lint.py                          # must return 0 ERROR

# 3. Rebuild derived artefacts
py -3 tools/build.py                   # refreshes graph + chunks + index + plugin/data/

# 4. Run the regression suite
py -3 tools/test.py                    # spawns 6 eval agents; must exit 0

# 5. Bump version (in plugin.json AND marketplace.json — they must match)
# 6. Commit, tag, push
git add wiki/ pakistan-regs-plugin/data/ pakistan-regs-plugin/.claude-plugin/
git commit -m "Add SBP_<NewCircular> — <one-line description>"
git tag -a v1.X.Y -m "..."
git push origin main --tags
```

That's the loop. The rest of this document explains *why* each step matters.

---

## Architecture refresher

```
wiki/                          ← source of truth (you edit here)
└── CLAUDE.md                  ← the schema; READ THIS BEFORE EDITING

raw/                           ← immutable cached PDFs (only the maintainer adds)

pakistan-regs-plugin/data/     ← shipped to teammates (auto-synced from wiki/)
├── wiki/                      ← copy of wiki/
├── chunks.jsonl               ← regenerated from wiki/ + raw/
└── graph.json                 ← regenerated from wiki/
```

You only ever edit `wiki/`. `tools/build.py` takes care of everything else.

---

## Adding a new regulation

### 1. Place the source

Drop the source PDF into `raw/` with a clear filename:

```
raw/<regulator>_<circular-number>_<year>.pdf
e.g.  raw/SBP_BPRD_C5_2026.pdf
```

If the source is HTML-only (a portal page), no `raw/` artefact is needed — the plugin will fall back to URL-only citation for that doc.

Update `raw/_download_log.json` to register the new file. (`tools/build.py`
reads this index when wiring chunks to source URLs.)

### 2. Create the wiki page

Filename: lowercase `doc_id` with underscores → hyphens.
`SBP_BPRD_C5_2026` → `wiki/sbp_bprd_c5_2026.md`.

Use this template (read `wiki/CLAUDE.md` for the full spec):

```markdown
---
doc_id: "SBP_BPRD_C5_2026"
name: "SBP — BPRD Circular No. 05 of 2026 (Example)"
type: regulation
regulator: "SBP"
reference: "BPRD Circular No. 05 of 2026"
date: "2026-04-15"
applicable_to: "Banks, DFIs, MFBs, Digital Banks, EMIs"
source_url: "https://www.sbp.org.pk/bprd/2026/C5.pdf"
status: "active"
supersedes: []
superseded_by: []
tags: [sbp, prudential, ...]   # only tags from CLAUDE.md taxonomy
---

# SBP — BPRD Circular No. 05 of 2026 (Example)

*One-line tagline in italics.*

## Purpose & Scope
A short paragraph stating what this regulation does and who issued it. Keep
to 2-4 sentences.

## Key Sections / Requirements
- Section A — short, specific obligation (e.g. "DBR cap of 50% for personal
  loans")
- Section B — …
- Section C — …

## Auto-Extracted Section Headings
*(Auto-populated by `tools/build.py` if the PDF is in `raw/`.)*

## Detected Keywords
*(Auto-populated.)*

## Related concepts
*(Auto-populated wikilinks — `tools/build.py` detects concepts and entities
from your body text. You can ALSO hand-add wikilinks under this heading;
the build script preserves manual entries.)*

## Related entities
*(Auto-populated wikilinks.)*

## See also
- [[OTHER_DOC_ID]] — short note on the relationship
```

**Required frontmatter fields** (lint will fail without them):
- `doc_id`, `name`, `regulator`, `reference`, `applicable_to`, `source_url`,
  `status`

**Status values**:
- `active` — default. Regulation is in force.
- `superseded` — replaced by a newer regulation; set `superseded_by` to its
  `doc_id`.
- `draft` — proposal / exposure draft, not yet in force.
- `stale-link` — content is authoritative but `source_url` is dead. See
  `MAINTENANCE.md`.

### 3. Update related pages

If the new regulation **supersedes** something, edit the older page:

```yaml
status: "superseded"
superseded_by: ["SBP_BPRD_C5_2026"]
```

`tools/build.py` enforces symmetry — if A says it supersedes B, B must say
it's superseded by A. Lint fails otherwise.

### 4. Run the build

```bash
py -3 lint.py                          # 0 ERROR / 0 WARN required
py -3 tools/build.py                   # refreshes graph, chunks, index, log
py -3 tools/test.py                    # 6 eval agents must all pass
```

`tools/build.py` is idempotent. Running it twice gives the same result.

### 5. Commit, version, push

Bump version per `CHANGELOG.md` rules:
- Adding a regulation → MINOR
- Fixing a typo / refreshing a URL → PATCH
- Changing citation schema / skill names → MAJOR

Edit BOTH version fields (they must match):
- `pakistan-regs-plugin/.claude-plugin/plugin.json` → `"version"`
- `pakistan-regs-plugin/.claude-plugin/marketplace.json` → `plugins[0].version`

Then:

```bash
git add wiki/ raw/_download_log.json pakistan-regs-plugin/ CHANGELOG.md
git commit -m "Add SBP BPRD C5/2026 — <topic>"
git tag -a v1.X.Y -m "Release notes here"
git push origin main --tags
```

Tell teammates the new version is out — they run `/plugin update pakistan-regs`.

---

## Editing an existing regulation

Same flow, smaller scope. If you're only fixing a typo or refreshing a
stale URL:

1. Edit `wiki/<doc_id>.md`
2. If the URL has changed, update `source_url` in frontmatter AND change
   `status` from `stale-link` back to `active`. Remove the entry from
   `MAINTENANCE.md`.
3. `py -3 lint.py && py -3 tools/build.py`
4. Bump PATCH version (1.0.1 → 1.0.2)
5. Commit, tag, push.

---

## Adding a new concept or entity

Concepts and entities are defined as catalog entries inside
`tools/build.py` (look for `CONCEPTS = [...]` and `ENTITIES = [...]`).

To add a concept:

1. Append a tuple to `CONCEPTS`:
   ```python
   ('new-concept-slug', 'New Concept Name',
    'Definition. Keep to 2-3 sentences.',
    ['alias1', 'alias2', 'alias3'],      # alias detection, case-insensitive
    ['tag1', 'tag2'],                    # only tags from CLAUDE.md taxonomy
    'Why this matters in 1-2 sentences.'),
   ```
2. Run `py -3 tools/build.py`. The script will:
   - Generate `wiki/concepts/<slug>.md` automatically
   - Scan all regulation pages and detect mentions
   - Wire up backlinks in both directions
3. If the new tags aren't in `wiki/CLAUDE.md`'s taxonomy, add them there too.
4. Bump MINOR version, commit, push.

To add a regulator or regulated-entity type: same pattern, append to
`ENTITIES`.

---

## What NOT to do

- **Don't hand-edit `chunks.jsonl`, `graph.json`, or
  `wiki/concepts/*.md` / `wiki/entities/*.md`.** They are regenerated by
  `tools/build.py`. Hand edits will be overwritten on next build.
- **Don't paraphrase a regulation without citing the chunk.** The skills'
  citation discipline depends on every claim being traceable to a real
  chunk in `chunks.jsonl`.
- **Don't invent a `doc_id`** in a `[[...]]` wikilink. Lint catches dangling
  wikilinks.
- **Don't add a tag** that isn't in the CLAUDE.md taxonomy. If you need a
  new one, add it to the taxonomy first.
- **Don't skip the lint or test runs** before pushing. Both are fast and
  catch real issues.

---

## When in doubt

- Read `wiki/CLAUDE.md` — it's the canonical schema.
- Look at how an existing regulation page is structured.
- Run `py -3 lint.py` — it tells you exactly what's wrong.
- Ask the maintainers in `#product-compliance` (Slack).

---

## Maintainers

- Initial author: AdalFi Product Team
- Contact: product-compliance@adalfi.ai
