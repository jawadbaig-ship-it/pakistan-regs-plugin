# MAINTENANCE.md — Known issues & refresh plan

This document tracks known limitations of the regulatory knowledge base so
maintainers (and curious teammates) understand what's intentionally
deferred rather than accidentally missing.

> **Important**: every issue below is *acknowledged in the data* — pages
> are tagged `status: stale-link`, the lint suppresses the warnings, and
> the affected curated content is still authoritative. Nothing here is a
> silent gap.

---

## A. Stale source URLs (17 pages)

These regulations have **curated wiki content that is accurate**, but their
original `source_url` returns HTTP 404. The SBP and FMU rotate their URL
schemes periodically; this list reflects the state of their sites in
mid-2026.

Each affected wiki page is tagged `status: stale-link` in its frontmatter,
which suppresses the `DEAD_SOURCE_URL` warning in `lint.py`.

### The 17 URLs

| Wiki page | Stale URL | Likely current location |
|---|---|---|
| `gop_sbp_act_1956` | `sbp.org.pk/l_frame/SBP_Act.pdf` | search: `site:sbp.org.pk "SBP Act"` |
| `gop_bco_1962` | `sbp.org.pk/l_frame/BCO_1962.pdf` | search: `site:sbp.org.pk "Banking Companies Ordinance"` |
| `gop_mfi_2001` | `sbp.org.pk/acd/MFI-Ordinance-2001.pdf` | search: `site:sbp.org.pk "MFI Ordinance"` |
| `gop_amla_2010` | `fmu.gov.pk/docs/AMLA-2010.pdf` | search: `site:fmu.gov.pk AMLA-2010` |
| `gop_cba_2015` | `sbp.org.pk/cpd/CBA-2015.pdf` | search: `site:sbp.org.pk "Credit Bureaus Act"` |
| `gop_eto_2002` | `sbp.org.pk/l_frame/ETO_2002.htm` | search: `site:sbp.org.pk "Electronic Transactions Ordinance"` |
| `sbp_psorules_2014` | `sbp.org.pk/psd/2014/PSO-Rules.htm` | PSD circulars index |
| `sbp_ifrs9` | `sbp.org.pk/bprd/2022/IFRS9.htm` | BPRD circulars index |
| `sbp_shariah` | `sbp.org.pk/ibd/index.asp` | IBD landing page (moved) |
| `sbp_raast` | `sbp.org.pk/PS/Raast.htm` | DFS Raast page |
| `sbp_sandbox` | `sbp.org.pk/psd/2025/Regulatory-Sandbox.htm` | PSD circulars index |
| `sbp_ecib_datafmt_dfs` | `dfs.sbp.org.pk/ecib/new-ecib.htm` | DNS failure — domain may have moved |
| `sbp_nfis` | `sbp.org.pk/Finc/index.htm` | financial-inclusion landing |
| `sbp_openbanking` | `sbp.org.pk/dfs/index.htm` | DFS landing |
| `sbp_policyrate` | `sbp.org.pk/m_policy/index.htm` | monetary policy landing |
| `sbp_housing` | `sbp.org.pk/hfd/index.htm` | HFD landing |
| `sbp_greenbanking` | `sbp.org.pk/smefd/circulars/2024/Green-Banking.htm` | SMEFD circulars index |

### Refresh procedure

For each entry above:

1. Run the suggested `site:` search on Google.
2. Open the new URL in a browser and verify it serves the intended
   document (or its successor).
3. Open `wiki/<doc_id>.md`, update `source_url` to the new URL, and change
   `status: stale-link` → `status: active`.
4. If applicable, drop the new PDF into `raw/` and update
   `raw/_download_log.json`.
5. Run `py -3 tools/build.py` to refresh chunks (so page deep links work).
6. Run `py -3 lint.py` — must remain `0 ERROR / 0 WARN`.
7. Remove the entry from the table above.
8. Bump PATCH version, commit with message `Refresh stale-link: <doc_id>`,
   push.

### Why we didn't fix these at v1.0.0

Each URL refresh requires opening the SBP/FMU portal and confirming the
new authoritative location. That's research work, not engineering work,
and was outside the scope of the initial build. The curated content in
each affected wiki page is hand-authored and remains authoritative —
only the link to the PDF is broken.

---

## B. Pages with sparse concept linking (25 pages)

Twenty-five regulation pages have empty automatic concept-detection
results because their body text is high-level and doesn't contain the
concept vocabulary (e.g., a press release that announces a circular
without explaining its contents; a primary act like the SBP Act 1956 that
sets up the central bank rather than detailing any specific compliance
mechanic).

Resolved partially in **v1.0.1** for primary legislation
(see `CHANGELOG.md`) by hand-adding `[[concepts/...]]` and
`[[entities/...]]` wikilinks at the bottom of those pages. The
`tools/build.py` script preserves hand-added wikilinks.

The remaining pages — mostly SECP press releases, Scribd 3rd-party copies,
and industry-news references — are intentionally left as-is. They link to
entities (via the regulator frontmatter) but not to concepts, because
linking a press release to "KFS" would be misleading: the press release
*announces* the rule but doesn't *contain* it. Better to traverse from the
press-release page to the actual circular via `## See also`.

If you'd like to enrich one of these pages, follow the wikilink convention:

```markdown
## Related concepts (manually curated)
- [[concepts/kfs|Key Fact Statement (KFS)]] — central obligation announced
- [[concepts/digital-lending|Digital Lending]] — broader area

## Related entities (manually curated)
- [[entities/secp|Securities and Exchange Commission of Pakistan (SECP)]]
```

---

## C. Pages without source PDF in `raw/`

Sources where SBP/SECP serves only HTML (no PDF) have no `#page=N` deep
link — citations fall back to the URL plus chunk index. This is normal
and documented in the `pakistan-regs-cite` skill's edge-case handling.

---

## D. Refresh cadence

Recommended cadence for periodic maintenance:

| Cadence | What to check |
|---|---|
| **Weekly** (lightweight) | Run `py -3 tools/test.py` to catch any regressions from upstream regulator updates. |
| **Monthly** (medium) | Run `py -3 lint.py`; refresh 1-2 stale URLs from Section A; redownload any source PDFs that have been updated in place. |
| **Quarterly** (heavy) | Re-scan SBP and SECP circulars indexes for any new circulars not yet in the wiki. Ingest per `CONTRIBUTING.md` flow. Re-verify the disclaimer in `LICENSE` with legal/compliance. |
| **On change** | Any time a teammate reports a wrong / outdated answer in production, treat as a high-priority correction: fix the source page, run the lint + build + test loop, tag a PATCH release. |

---

## E. Reporting an issue

If you spot a content issue, edge case, or installation problem:

1. Reproduce the issue with `/reg-search` or `/reg-validate` in your
   Claude Code session.
2. Note the exact prompt, the (possibly wrong) output, and the expected
   output.
3. File an issue (or message `#product-compliance`) with:
   - The prompt
   - The actual answer (including any cited `[[doc_id]]` and `chunk_id`)
   - The expected answer and which source supports it
4. Maintainers will reproduce, fix the wiki page, run the test suite, and
   ship a PATCH release.
