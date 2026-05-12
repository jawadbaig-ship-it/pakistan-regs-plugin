---
description: Validate a PRD, spec, user story or feature against Pakistan lender regulations. Produces a structured report with blockers, warnings, and citations.
---

Use the **pakistan-regs-validate-prd** skill to review the artefact described / pasted below.

Follow the skill's 6-step flow:
1. Read `data/wiki/CLAUDE.md` and `data/wiki/index.md`.
2. Extract the 8-point PRD profile (entity type, product, segment, channel, onboarding, data signals, money flow, algorithmic decisions).
3. Walk the applicable rule areas listed in the skill's table.
4. Produce the verdict in the exact structure given (PRD profile → Applicable rule set → 🔴 BLOCKERS → 🟡 WARNINGS → 🔵 INFO → Out of scope → Open questions).
5. Cite every finding with `[[doc_id]]`, section anchor, deep-linked PDF, verbatim quote, and `chunk_id`.
6. Pattern-match against the 10 known-bad patterns.

Artefact to validate:

$ARGUMENTS
