---
name: log
description: "Append-only changelog of wiki operations"
type: log
---

# Wiki Changelog

_Newest entries on top._

## [2026-05-12] build | tools/build.py rebuild
- Refreshed wikilinks + concept/entity pages
- Rebuilt graph.json (174 nodes / 671 edges)
- Refreshed chunks.jsonl and index.md

## [2026-05-11] lint-fix | apply 5 quality fixes
- Added `status: active` to 98 regulation pages
- Marked 17 pages with `status: stale-link` (source URL 404)
- Resolved 2 alias collisions (banking-mohtasib, digital-lending vs DLA)
- Extended CLAUDE.md tag taxonomy with detected-keyword tags
- Backfilled 2 reporting-entity links (fmu_portal, gop_un_sanctions)

## [2026-05-11] kg-fix | plural-form detection
- Detection regex now allows trailing `s` (plurals) — earlier "PEPs", "DLAs", "EMIs", "MFBs", "STRs" etc. were missed
- Re-detected and rewrote 39 concept + 20 entity pages
- Refreshed wikilink blocks on 115 regulation pages
- Rebuilt graph: 174 nodes, 676 edges (was 510)

## [2026-05-11] kg-build | knowledge-graph synthesis pass
- Generated 39 concept pages under `concepts/`
- Generated 20 entity pages under `entities/`
- Detected and inserted wikilinks across 115 regulation pages
- Built `graph.json` (174 nodes / 510 edges) and `graph.graphml`
- Regenerated content-organized `index.md`
- Authored `CLAUDE.md` schema for future ingest/query/lint operations

## [2026-05-10] ingest | initial corpus
- Source 1: Pakistan_Regulatory_Compliance_Knowledge_Base.xlsx (70 regulations)
- Source 2: Pakistan Lender Regulatory Knowledge Compilation.docx (49 additional URLs)
- Built consolidated workbook `Pakistan_Lender_Regulatory_KB_Consolidated.xlsx` with 119 master rows
- Tier-1 enrichment: 15 priority documents deeply summarised
- Tier-2 enrichment: 25 documents added
- Tier-3 enrichment: 75 documents added (115 total Enriched Details rows)
- Built initial `wiki/` (115 .md), `raw/` cache (94/112 PDFs), `chunks.jsonl` (3,217 chunks)

