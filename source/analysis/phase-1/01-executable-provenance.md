# 01 — Executable provenance

**Date:** 2026-11-16
**Source location:** `D:\Digiexam\` (installed DigiExam client)
**Method:** PowerShell `Get-FileHash`, `Get-AuthenticodeSignature`,
`System.Diagnostics.FileVersionInfo`, direct PE header read. Read-only; binaries were
never executed for this analysis.

## 1. File inventory

| File | Size (bytes) | Size | PE machine | Role (observed) |
|---|---|---|---|---|
| `digiexam.exe` | 44,482,696 | 42.4 MiB | `0x8664` (x86-64) | Tauri 2.x application host (UI + IPC client) |
| `service.exe` | 18,614,440 | 17.8 MiB | `0x8664` (x86-64) | Windows service `DigiexamWindowsService` (lockdown executor) |
| `toolbox-ipc-proxy.exe` | 1,211,904 | 1.2 MiB | `0x8664` (x86-64) | IPC broker between client and service |

Other content in the install directory: `Uninstall Digiexam.lnk` and two empty
directories created by the researcher (`Digiexam_Modified`, `Source`). **No** Electron
redistributables (`*.pak`, `ffmpeg.dll`, `libEGL.dll`, `resources\app.asar`, `locales\`)
are present — the directory layout itself is inconsistent with an Electron install.

## 2. Hashes (SHA-256)

| File | SHA-256 |
|---|---|
| `digiexam.exe` | `047BA68F640FD27F86C4D43DB3E90DC4BBEDBB7C126038BAD15EFBCC04E85491` |
| `service.exe` | `3EE5A430D6BF400245EDCCE17F754D39BB1C980A27E17F7B491CAB96554D907E` |
| `toolbox-ipc-proxy.exe` | `05674FC5BBEF51E303710E9AD6A1FCD14CABFA9549EC7BB72D90C7B6D3D0022B` |

## 3. Version resources

| File | ProductName | FileVersion | ProductVersion | CompanyName |
|---|---|---|---|---|
| `digiexam.exe` | Digiexam | 26.4.7 | 26.4.7 | digiexam |
| `service.exe` | — (empty) | — | — | — |
| `toolbox-ipc-proxy.exe` | — (empty) | — | — | — |

The empty version resources on `service.exe` and `toolbox-ipc-proxy.exe` are consistent
with Rust/Cargo builds that do not embed a `VERSIONINFO` resource; `digiexam.exe` gets
its resources from the Tauri bundler.

## 4. Authenticode signatures (observed, not cryptographically re-verified beyond status)

All three files: **`Status: Valid`**.

| File | Signer subject |
|---|---|
| `digiexam.exe` | `CN=DigiExam Solutions Sweden AB, O=DigiExam Solutions Sweden AB, S=Stockholms län, C=SE, SERIALNUMBER=556869-1504` (+ EV-style private-organization OIDs) |
| `service.exe` | same |
| `toolbox-ipc-proxy.exe` | `CN=DigiExam Solutions Sweden AB, O=DigiExam Solutions Sweden AB, S=Stockholms län, C=SE` |

**Inference:** all three components ship from the same vendor identity; the service is
not a third-party driver/helper dropped by another party.

## 5. Toolchain identification (marker counts, direct regex over full-file ASCII + UTF-16LE)

| Marker | `digiexam.exe` | `service.exe` | `toolbox-ipc-proxy.exe` |
|---|---|---|---|
| `tauri` | **404** | 0 | 0 |
| `WebView2` | 20 | 0 | 0 |
| `lockdown` | 1140 | 968 | 24 |
| `Electron` (case-sensitive) | 2 | 1 | 0 |
| `node.dll` | 0 | 0 | 0 |
| `BSJB` (.NET CLR metadata magic) | 0 | 0 | 0 |
| `GetLockdownConfig` | 8 | 8 | 0 |

**Observations:**
- `BSJB = 0` in every binary → no .NET/CLR metadata; dnSpy-based managed analysis is
  not applicable (matches phase-1 finding).
- `node.dll = 0` → no Node.js runtime linkage.
- The 2 `Electron` hits in `digiexam.exe` were resolved in phase 1 to the substrings
  "Electron**ic**" (word-list blob, file offset ≈ 34,276,074) and "Secure **Electronic**
  Transactions" (X.509 OID name, ≈ 36,344,983) — i.e. false positives, not the Electron
  framework.

**Conclusion:** native x86-64 Rust/Tauri binaries; **no Electron/Node/.NET layer**.
