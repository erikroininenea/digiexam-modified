# Recovery status — phase 3

**Date:** 2026-11-16 · **Status:** extraction **partially executed, then cancelled by
the operator** (project owner decision, 2026-11-16). This file records honestly what
was and was not recovered. Nothing in this file is fabricated to satisfy the phase-3
completion criteria.

## 1. What was completed

| Item | Result |
|---|---|
| Asset-path enumeration (full binary, `digiexam.exe`) | **Done.** 108 distinct path-like strings; 26 under `/assets/`: 18 × `KaTeX_*.woff2` (SIL OFL — redistributable), 4 × `Metrik-*.woff` (proprietary), 1 × `TitilliumWeb-*.ttf` (proprietary, Google Fonts license — bundled copy not redistributable by this project without review), 1 × main JS bundle, 1 × CSS, 1 × KaTeX_Main-Bold variant |
| Verified anchor extractions (phase-1/2 provenance chain) | `index.html` @30,807,145 (1,167 B, SHA-256 `5C2300BC…E01EE76`), `index-COO2fUbz.js` @28,509,689 (10,547,069 B, `7B87FB3A…33AA429`), `index-CvNvudRa.css` @31,489,343 (144,919 B, `692F5112…3977F73`) — re-extracted from the signed binary and hash-matched byte-for-byte (see [`../recovered-assets/MANIFEST.md`](../recovered-assets/MANIFEST.md)) |
| Raw string-evidence locations (lockdown components) | **Done and published** — [`../analysis/14-lockdown-raw-evidence.md`](../analysis/14-lockdown-raw-evidence.md): 30 handler modules + 32 support modules + plugin/app-shell paths pinned by byte-exact offset in both `digiexam.exe` and `service.exe` |
| Analysis documents | [`../analysis/12-protobuf-schemas.md`](../analysis/12-protobuf-schemas.md) and [`../analysis/13-lockdown-architecture-boundaries.md`](../analysis/13-lockdown-architecture-boundaries.md) |
| Publication policy | [`recovered-source/.gitignore`](../recovered-source/.gitignore) — proprietary artifacts excluded from git |
| Local preservation | frontend bundle/CSS/HTML, three string dumps, main executable icon (all gitignored, on disk) |

## 2. What was cancelled (operator decision)

- The **full brute-force brotli scan** of `digiexam.exe` (44,482,696 offset
  candidates) was cancelled after ~0–1 M offsets had been processed. All partial
  blob output was deleted; the asset-path manifest produced by the scan was removed
  as well. The scan produces no results until the final mapping step, so no data was
  lost beyond the enumeration summary above (captured from the scan's console output
  before cancellation).

## 3. Phase-3 completion-criteria status

| Criterion | Status |
|---|---|
| 1. Actual files recovered | Partial — 3 verified anchors + icon preserved **locally**; 0 files published from the cancelled scan |
| 2. File counts by format | Not applicable to the cancelled scan (see §2); enumerable components: 18 woff2 (publishable), 4 woff + 1 ttf (proprietary), 1 js + 1 css + 1 html (proprietary) |
| 3. SHA-256 verification | Done for the 3 anchors (§1); not applicable to cancelled scan |
| 4. Original vs reconstructed | **0 original source files exist in the binaries** (release build; no source maps). Everything recoverable is *extracted production output*; the beautified copy is *reconstructed* and stays local |
| 5. Excluded from publication | Frontend bundle/CSS/HTML, beautified reconstruction, blobs, icons — see [`recovered-source/.gitignore`](../recovered-source/.gitignore) and [`TOOLING-SNIPPETS.md`](TOOLING-SNIPPETS.md) §"Not included" |
| 6. Tools / MCP servers used | Documented in [`../analysis/11-tooling-and-methodology.md`](../analysis/11-tooling-and-methodology.md) + [`TOOLING-SNIPPETS.md`](TOOLING-SNIPPETS.md) |
| 7. Remaining limitations | No brotli-stream magic bytes ⇒ full-scan approach required (costly); no source maps; no PDBs ⇒ string offsets cannot be mapped to function boundaries; KaTeX fonts are the only clearly redistributable asset family and were **not** extracted before cancellation |

## 4. Resumption

If the operator lifts the cancellation: re-run
`%TEMP%\opencode\phase3\extract_assets.js` (full scan + content sniffing), then map
blobs → asset paths, extract the 18 SIL-OFL KaTeX fonts with license text, and
finalize the full `EXTRACTION-REPORT.md` + `SHA256-MANIFEST.txt`. Until then, this
status file is the authoritative record.
