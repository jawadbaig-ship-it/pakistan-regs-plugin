# Changelog

All notable changes to the `pakistan-regs` plugin are recorded here.
The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)
and the plugin uses [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Versioning rules for this plugin:
- **MAJOR** (`x.y.z` → `(x+1).0.0`) — breaking change to the citation format,
  skill names, slash-command names, or `chunks.jsonl` schema.
- **MINOR** (`x.y.z` → `x.(y+1).0`) — new regulation, new skill, new
  command, or new concept/entity page.
- **PATCH** (`x.y.z` → `x.y.(z+1)`) — typo fix, citation refresh, lint fix,
  stale-link URL update, or any change that doesn't alter the externally-
  observable contract.

---

## [Unreleased]

### Planned
- MCP server variant (same `data/` directory, additional consumption
  surface for Cursor / Claude Desktop / CI agents)
- Refresh of the 17 stale-link source URLs (see `MAINTENANCE.md`)
- Hand-curated concept links on 25 high-level pages where automatic
  detection found nothing meaningful

---

## [1.0.1] — 2026-05-13

### Added
- `LICENSE` (proprietary — internal AdalFi use only)
- `CHANGELOG.md` (this file)
- `CONTRIBUTING.md` (contribution workflow, lint expectations, version-bump
  rules)
- `MAINTENANCE.md` (known issues: 17 stale source URLs, 25 lightly-linked
  pages, planned refresh cadence)
- Hand-curated `## Related concepts` and `## Related entities` blocks on
  primary legislation pages (`GoP_SBP_Act_1956`, `GoP_BCO_1962`,
  `GoP_MFI_2001`, `GoP_PSEFT_2007`, `GoP_AMLA_2010`, `GoP_CBA_2015`,
  `GoP_PECA_2016`, `GoP_PDPB_2023`, `GoP_ETO_2002`, `GoP_Companies_Act`)
  to compensate for sparse keyword density in statutory language.
- `tools/test.py` — automated regression runner that re-spawns the 6 eval
  prompts (with-skill + baseline) and grades against `evals/evals.json`.
  Returns non-zero exit status if any with-skill assertion fails.

### Changed
- Plugin version bumped to `1.0.1` in `.claude-plugin/plugin.json` and
  `.claude-plugin/marketplace.json`.

### Fixed
- None (no functional regressions; this is a polish/maintainability release.)

---

## [1.0.0] — 2026-05-12

Initial release.

### Added
- **Knowledge base**
  - 115 regulation pages (SBP, SECP, FMU, NADRA, MoITT, PTA, MoF, PBA, GoP
    primary legislation)
  - 39 cross-cutting concept pages (KFS, eCIB, CDD, EDD, AML/CFT, STR, CTR,
    Outsourcing, Cloud, Sandbox, Whitelist, Digital Lending, Nano-Lending,
    Raast, Branchless Banking, Fair Debt Collection, BC&FRF, CGHM,
    Banking Mohtasib, Basel III, IFRS-9, Shariah Governance,
    Cybersecurity, BCP/DR, Tech Risk, Classification & Provisioning,
    Capital Adequacy, Data Protection, …)
  - 20 entity pages (SBP, SECP, FMU, NADRA, PTA, MoITT, MoF, PBA, Banking
    Mohtasib · Bank, DFI, MFB, Digital Bank, EMI, NBFC, PSP, PSO, Credit
    Bureau, DLA, Reporting Entity)
  - 3,217 embedding-ready chunks in `data/chunks.jsonl`
    - 2,771 chunks with `#page=N` PDF deep links
    - 1,566 chunks with pre-computed `section_anchor` (§4.2 …, Section II
      …, Regulation 1 — CDD, etc.)
  - Knowledge graph (`data/graph.json` + `data/graph.graphml`): 174 nodes
    (115 reg + 39 concept + 20 entity), 671 edges
    (`issued_by` · `applies_to` · `addresses` · `references`)
- **Three skills**
  - `pakistan-regs-search` — natural-language search with citations
  - `pakistan-regs-validate-prd` — compliance review of PRDs/specs with
    BLOCKER / WARNING / INFO verdict
  - `pakistan-regs-cite` — publication-quality citation block (Variant A
    inline + Variant B block) with chunk-ID verification and fabrication
    refusal
- **Three slash commands**
  - `/reg-search <question>`
  - `/reg-validate <PRD or paste>`
  - `/reg-cite <doc_id or topic>`
- **Marketplace manifest** (`.claude-plugin/marketplace.json`) so the repo
  installs via `/plugin marketplace add` → `/plugin install
  pakistan-regs@adalfi-regs`
- **Evaluation harness** (`evals/evals.json`): 6 prompts × 24 assertions
  - Initial scores: with-skill 24/24 auto-pass, baseline 11/24, delta +13
- **Schema documentation** (`data/wiki/CLAUDE.md`) defining the three-layer
  architecture, page templates, citation discipline, ingest / query / lint
  workflows, and tag taxonomy

### Provenance
Built from two source documents:
- `Pakistan_Regulatory_Compliance_Knowledge_Base.xlsx` — 70 master
  regulations with regulator, category, applicable-to, URLs.
- `Pakistan Lender Regulatory Knowledge Compilation.docx` — 49 additional
  URLs cited in a narrative compilation.

### Known limitations at v1.0.0
- 17 source URLs returned HTTP 404 at build time (SBP / FMU URL changes
  since the source spreadsheet was compiled). Affected pages carry the
  curated content and are tagged `status: stale-link`. See `MAINTENANCE.md`
  for the list and refresh plan.
- 25 regulation pages (mostly primary acts, press releases, and 3rd-party
  copies) have no automatically-detected concept wikilinks. The concept
  vocabulary genuinely doesn't appear in their high-level prose. Resolved
  in 1.0.1 for primary acts; press releases and 3rd-party copies remain
  as-is intentionally.

---

## Versioning checklist (for maintainers)

When you cut a release:

- [ ] All wiki pages pass `py -3 lint.py` with `0 ERROR / 0 WARN`
- [ ] `py -3 tools/build.py` runs to completion (graph + chunks regenerated,
      plugin `data/` synced, lint clean)
- [ ] `py -3 tools/test.py` returns exit 0 (all 6 eval prompts pass)
- [ ] Bump `version` in `.claude-plugin/plugin.json` AND
      `.claude-plugin/marketplace.json` (must match)
- [ ] Add a new section at the top of this file
- [ ] `git commit && git tag -a vX.Y.Z -m "..." && git push --tags`
- [ ] Tell teammates the version is available: they `/plugin update
      pakistan-regs` to refresh
