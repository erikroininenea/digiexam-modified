# Recovered assets — MANIFEST

All artifacts below were recovered **read-only** from the installed, signed DigiExam
client at `D:\Digiexam\`. Files are **preserved locally** in this directory; the
proprietary ones are excluded from the public repository (see
[`../documentation/legal-and-scope.md`](../documentation/legal-and-scope.md)).

## 1. Source binaries (inputs, not stored in this repo)

| File | Size | SHA-256 | Signature |
|---|---|---|---|
| `digiexam.exe` (v26.4.7) | 44,482,696 B | `047BA68F640FD27F86C4D43DB3E90DC4BBEDBB7C126038BAD15EFBCC04E85491` | Valid, DigiExam Solutions Sweden AB |
| `service.exe` | 18,614,440 B | `3EE5A430D6BF400245EDCCE17F754D39BB1C980A27E17F7B491CAB96554D907E` | Valid, DigiExam Solutions Sweden AB |
| `toolbox-ipc-proxy.exe` | 1,211,904 B | `05674FC5BBEF51E303710E9AD6A1FCD14CABFA9549EC7BB72D90C7B6D3D0022B` | Valid, DigiExam Solutions Sweden AB |

## 2. Recovered frontend assets (from `digiexam.exe`)

Tauri release assets are **brotli-compressed without a magic signature**; streams were
located by offset scanning + successful `zlib.brotliDecompressSync` validation.

| Local path | Offset in `digiexam.exe` | Size (decompressed) | SHA-256 | Nature | In git? |
|---|---|---|---|---|---|
| `frontend/index.html` | 30,807,145 | 1,167 B | `5C2300BCD2FA739F4A530884786A27144C4A111FF2B33C5043EA63280E01EE76` | Vite HTML shell referencing the bundle/CSS | no |
| `frontend/assets/index-COO2fUbz.js` | 28,509,689 | 10,547,069 B | `7B87FB3A311EF6378B0AD45A0A23108FA6FCAB1BACC75B4A0222121F633AA429` | **Minified build output** (Vite/esbuild, React + Redux Toolkit). NOT original source code | no |
| `frontend/assets/index-CvNvudRa.css` | 31,489,343 | 144,919 B | `692F5112AE6C94ABCC34B1E23CF7E5E9496FA297A02C0B67C922CEFDE3977F73` | Minified CSS | no |

**Verification:** all three were re-extracted fresh from the signed binary in phase 2
and hash-matched byte-for-byte. Reproduction script:
[`../analysis/phase-1/05-tooling-and-reproducibility.md`](../analysis/phase-1/05-tooling-and-reproducibility.md).

**Status of recovered code:** build artifacts recovered from a release binary. They
are *not* DigiExam's original source code, and no claim is made otherwise. Third-party
components identifiable inside the bundle include React, Redux Toolkit, and
KaTeX font assets (`KaTeX_*.woff2` referenced in the asset table) — vendor code and
third-party dependencies are intermixed by the bundler and are **not** separated here.

## 3. String dumps (full-file ASCII/UTF-16 extractions)

| Local path | Size | SHA-256 | Source |
|---|---|---|---|
| `strings/strings_digiexam.txt` | 9,592,327 B | `856DCA7A7D277E0C8372EE8BCDD9FF021490BADC8803DDAE3136CDBB40B96BAD` | `digiexam.exe` |
| `strings/strings_service.txt` | 3,693,757 B | `C4362613B33EDBDF4A496C9B6B0053E6B8790800CA0AC2E108053691CF6712C4` | `service.exe` |
| `strings/strings_toolbox.txt` | 420,276 B | `93355974A2418B748637D06D724438120F12ED96AB9C1DAAB48FB1F5EB78EC0C` | `toolbox-ipc-proxy.exe` |

Phase-1 report (researcher-authored, preserved verbatim in
`../documentation/digiexam_lockdown_rapport.md`):
SHA-256 `DBC9C614798295886DBF643D17F990BCBC59BEB0BD0933D6B37244B2DD53B597`.

## 4. Prior-session working files (volatile, documented not preserved)

Phase-1 helper scripts and REA outputs lived in `%TEMP%\opencode\`
(`brotli_scan.js`, `invoke_scan.js`, `acl_extract.js`, `proctor_scan.js`,
`rea_result.json`, `rea_bundle`, …). They are documented for reproducibility in
[`../analysis/phase-1/05-tooling-and-reproducibility.md`](../analysis/phase-1/05-tooling-and-reproducibility.md);
none are published.
