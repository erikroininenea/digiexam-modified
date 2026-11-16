# digiexam-modified

Reverse-engineering and architecture research on the **DigiExam for Windows** client
(v26.4.7), performed strictly by **read-only static analysis**.

> **Status: research/documentation only.** No binary was patched, no security
> mechanism was bypassed, and no bypass technique is documented here. See
> [`source/documentation/legal-and-scope.md`](source/documentation/legal-and-scope.md).

## Headline findings

| Question | Answer (verified) |
|---|---|
| Underlying architecture | **Tauri 2.x (Rust) + Microsoft Edge WebView2** frontend (React/Redux/Vite bundle) |
| Electron / Node.js components | **None.** No `node.dll`, no `app.asar`, no Electron frameworks. The 2 `Electron` strings in `digiexam.exe` are false positives ("Electron**ic**" in a word-list blob, "Secure **Electron**ic Transactions" in an X.509 OID table) |
| Chromium components | Present **only via the WebView2 runtime** (Chromium-based Edge webview); no Chromium/Node is bundled in the app itself |
| Lockdown engine | `service.exe` runs as Windows service **`DigiexamWindowsService`** (Rust, `lockdown-windows-service` / `lockdown-core` / `lockdown-windows`), driven by server-supplied operation definitions over gRPC + named-pipe IPC |
| Code signing | All three binaries validly signed by **DigiExam Solutions Sweden AB** (org.nr 556869-1504) |

## Repository layout

```
source/
├── analysis/                 # Analytical reports (this phase + phase 1)
│   ├── 01-executable-provenance.md
│   ├── 02-platform-identification.md
│   ├── 03-component-architecture.md
│   ├── 04-frontend-ipc-surface.md
│   └── 05-tooling-and-reproducibility.md
├── documentation/
│   ├── methodology.md                # Methods, tools, verification chain
│   ├── legal-and-scope.md            # Scope, ethics, copyright handling
│   └── digiexam_lockdown_rapport.md  # Phase-1 report (Swedish, preserved verbatim)
└── recovered-assets/
    ├── MANIFEST.md           # Provenance, hashes, extraction method
    ├── frontend/             # Preserved locally; large proprietary files NOT pushed
    └── strings/              # String dumps of the three binaries (preserved locally)
```

## Why the large extracted files are not in git

The extracted frontend bundle (`index-COO2fUbz.js`, 10.5 MB) and CSS are
**DigiExam's proprietary code**, recovered from a signed commercial binary. They are
preserved on the researcher's disk and fully documented (sizes, SHA-256, byte offsets,
reproduction script) in [`source/recovered-assets/MANIFEST.md`](source/recovered-assets/MANIFEST.md),
but are excluded from the public repository via `.gitignore` to avoid redistributing
copyrighted material. Remove the exclusions only if you hold permission to redistribute.

## Provenance quick reference

| Artifact | SHA-256 (first 16) | Origin |
|---|---|---|
| `digiexam.exe` 26.4.7 | `047BA68F640FD27F` | `D:\Digiexam\digiexam.exe` |
| `service.exe` | `3EE5A430D6BF4002` | `D:\Digiexam\service.exe` |
| `toolbox-ipc-proxy.exe` | `05674FC5BBEF51E3` | `D:\Digiexam\toolbox-ipc-proxy.exe` |

Full hashes: [`source/analysis/01-executable-provenance.md`](source/analysis/01-executable-provenance.md).
