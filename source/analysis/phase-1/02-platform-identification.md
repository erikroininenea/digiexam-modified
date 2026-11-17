# 02 — Platform identification: Tauri, not Electron

**Question posed by the research plan:** *does the executable contain
Electron/Chromium/Node.js components?*

## 1. Verdict

| Hypothesis | Result | Basis |
|---|---|---|
| Electron app | **No** | No Electron binaries/DLLs in the install directory, no `resources\app.asar`, no `node.dll`, no Electron version strings; the only `Electron` substrings are unrelated English/OID text (phase-1 offsets 34,276,074 and 36,344,983) |
| .NET / WPF / WinForms | **No** | `BSJB` CLR-metadata magic absent from all three PEs; dnSpy cannot load them (verified in phase 1 and re-verified here) |
| **Tauri 2.x + WebView2** | **Yes** | 404 `tauri` hits and 20 `WebView2` hits in `digiexam.exe`; phase 1 resolved exact crate versions below |

Version evidence from phase-1 string analysis of `digiexam.exe`:

- `tauri-2.10.3\src\ipc\command.rs`
- `tauri-plugin-updater-2.10.1`
- `tauri-plugin-deep-link-2.4.7`
- `tauri-plugin-store-2.4.2`
- `__TAURI_INTERNALS__` (inline JS bootstrap in the webview)

**Answer to "Chromium components":** yes, *indirectly* — the UI runs in Microsoft Edge
**WebView2**, which is Chromium-based. But the application does **not** bundle
Chromium or Node.js the way Electron does; it consumes the OS-provided WebView2
runtime. This distinction matters for every architectural conclusion downstream.

## 2. Component mapping (Electron concepts → DigiExam reality)

Since the original research plan assumed Electron, the mapping is documented
explicitly:

| Electron concept | DigiExam v26.4.7 equivalent |
|---|---|
| Main process (Node.js) | **Rust core** in `digiexam.exe` (`native\apps\digiexam\src\lib.rs`, `command\*.rs`) |
| Renderer process | WebView2 webview hosting a Vite/React bundle (`/assets/index-COO2fUbz.js`) |
| `contextBridge` / preload | Tauri IPC command registry + capability/ACL system (Tauri v2 permissions) |
| Native Node modules | **None exist.** Equivalent functionality is implemented as **Tauri plugins in Rust**: `lockdown-lite`, `webview-manager`, `io`, `battery`, `keyboard-layout`, `store`, `updater`, `deep-link` |
| `electron-builder` packaging | Tauri NSIS/MSI bundler (single monolithic exe + separate service) |
| DevTools | `internal_toggle_devtools` command exists in the ACL allow-list (phase 1, §8.2); whether the release webview exposes devtools is governed by webview configuration, **not verified at runtime** |

## 3. Frontend stack (from the recovered bundle)

- Build chain: **Vite/esbuild** (hashed asset names, minified single-file bundle)
- Framework: **React + Redux Toolkit** (thunk-based exam/lockdown orchestration)
- Runtime platform constant (phase 1, §7.5):
  `PLATFORM = { NATIVE: "native", WEB: "web", WASM: "wasm" }` — the same React app
  also ships as a browser/WASM variant; in this Windows build the **native** mode is
  active and lockdown flows through the `lockdown-lite` Tauri plugin.

## 4. WebView2 runtime dependency

**Observation:** `digiexam.exe` references the WebView2 loader; the runtime itself
(`msedgewebview2.exe` et al.) is not part of the `D:\Digiexam` directory.

**Inference (standard WebView2 distribution model):** the app depends on an
OS-installed Evergreen WebView2 runtime. **Unknown:** whether the vendor also ships a
fixed-runtime variant of the installer; this analysis covers the installed state only.
