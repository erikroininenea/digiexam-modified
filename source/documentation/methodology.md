# Methodology

## Phases

| Phase | Date | Focus | Output |
|---|---|---|---|
| 1 | prior session | Native string/constant analysis of all three binaries; dnSpy feasibility check; frontend asset extraction; frontend IPC mapping; REA CLI runs | `digiexam_lockdown_rapport.md` (Swedish, preserved verbatim in this directory) |
| 2 | 2026-11-16 | Independent re-verification of platform identification; provenance/hash chain; repository organization and publication | the `analysis/` reports and `recovered-assets/MANIFEST.md` in this repository |

## Method (both phases)

1. **File-format identification first** — PE machine type, CLR metadata presence,
   version resources, Authenticode signatures, install-directory layout.
2. **Read-only inspection** — binaries were hashed, string-dumped and pattern-scanned;
   **never executed, never patched, never modified**. No unpacking beyond reading
   bytes from the signed files.
3. **Format-aware asset recovery** — Tauri release assets are brotli-compressed
   without a magic signature; streams were located by scanning candidate byte offsets
   and validating successful decompression (~30,000 offsets ≈ 16 ms per pass).
4. **Cross-verification** — every headline claim was re-derived independently in
   phase 2 (marker counts, PE checks, fresh re-extraction + hash comparison) rather
   than trusting phase-1 output.
5. **Provenance discipline** — every artifact records: original location, SHA-256,
   size, extraction method, and the binary offset it came from. Recovered
   vendor code is never presented as "source code" — it is minified build output.

## Tools

PowerShell 5.1 (hashing, signatures, PE headers), Node.js 26 (`zlib` brotli,
hashing), git, GitHub MCP, REA CLI fallback (`npx -y rea-agents@6.9.0`).
dnSpy MCP was connected but is **not applicable** (no .NET metadata — see
[`../analysis/01-executable-provenance.md`](../analysis/01-executable-provenance.md)).

## Verification chain (reproducible)

```
D:\Digiexam\digiexam.exe                 SHA-256 047BA68F…E85491  (Authenticode: DigiExam Solutions Sweden AB)
   └─ offset 30807145  → brotli → index.html                  SHA-256 5C2300BC…EE76  ✅ re-extraction match
   ├─ offset 28509689  → brotli → assets/index-COO2fUbz.js    SHA-256 7B87FB3A…3429  ✅ re-extraction match
   └─ offset 31489343  → brotli → assets/index-CvNvudRa.css   SHA-256 692F5112…9F73  ✅ re-extraction match
```

Reproduction scripts: [`../analysis/05-tooling-and-reproducibility.md`](../analysis/05-tooling-and-reproducibility.md).

## Evidence conventions

- **Observation** — directly read from the artifacts (string at offset, header field,
  hash match).
- **Inference** — conclusion drawn from observations; the inference itself is labeled.
- **Unknown / not verified** — explicitly recorded rather than guessed
  (e.g. runtime devtools exposure, fixed-runtime installer variants, live operation
  distribution).
