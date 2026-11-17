# 05 — Tooling, reproducibility & verification log

All work in both phases was **read-only static analysis**. This document records the
toolchain and the exact reproduction steps so any claim can be re-verified.

## 1. Toolchain

| Purpose | Tool | Notes |
|---|---|---|
| Hashing, version info, Authenticode | PowerShell (`Get-FileHash`, `Get-AuthenticodeSignature`, `FileVersionInfo`) | Windows PowerShell 5.1 — note: no `??` operator |
| PE header check | `System.IO.BinaryReader` (e_lfanew → `machine`) | all three binaries: `0x8664` |
| String dumps | Sysinternals-style `strings` equivalents → `strings_digiexam.txt` (9,592,327 B), `strings_service.txt` (3,693,757 B), `strings_toolbox.txt` (420,276 B) | preserved under `recovered-assets/strings/` |
| Marker counting | `Regex.Matches` over full-file ASCII + UTF-16LE decodes | see script below |
| Asset extraction | Node.js `zlib.brotliDecompressSync` + offset scanning | Tauri release builds brotli-compress embedded assets; no magic signature exists, so streams are located by scanning candidate offsets (≈30,000 offsets ≈ 16 ms) |
| Managed-code check | dnSpy (via MCP) — **not applicable**: no `BSJB` CLR metadata in any binary | documented in phase-1 §6 |
| JS bundle analysis | REA CLI fallback `npx -y rea-agents@6.9.0 analyze-javascript-application --json` | REA MCP server not registered in either session; phase-1 §7.6 documents two runs: default heap OOM on the 10.5 MB bundle (`parse_status: "skipped"`); second run with `--max-heap-mb 4096` hit the 300 s per-source worker timeout. Electron-facing fields all reported **0**, independently confirming "not Electron" |

## 2. Marker-count script (this phase, verified)

```powershell
$out = foreach ($p in @('D:\Digiexam\digiexam.exe','D:\Digiexam\service.exe','D:\Digiexam\toolbox-ipc-proxy.exe')) {
  $b = [System.IO.File]::ReadAllBytes($p)
  $ascii = [System.Text.Encoding]::ASCII.GetString($b)
  $uni   = [System.Text.Encoding]::Unicode.GetString($b)
  [PSCustomObject]@{
    Bin     = (Split-Path $p -Leaf)
    tauri   = ([regex]::Matches($ascii,'tauri').Count   + [regex]::Matches($uni,'tauri').Count)
    WebView2= ([regex]::Matches($ascii,'WebView2').Count + [regex]::Matches($uni,'WebView2').Count)
    lockdown= ([regex]::Matches($ascii,'lockdown').Count + [regex]::Matches($uni,'lockdown').Count)
    Electron= ([regex]::Matches($ascii,'Electron').Count + [regex]::Matches($uni,'Electron').Count)
    BSJB    = ([regex]::Matches($ascii,'BSJB').Count)
  }
}
$out | Format-Table -AutoSize
```

**Pitfall recorded for future runs:** wrapping `Regex.Matches(...).Count` inside a
ScriptBlock and calling `.Invoke(...).Count` silently returns the *wrapper's* count
(1) — first run of this check produced identical bogus counts for all binaries and was
redone with direct inline `Regex.Matches`.

## 3. Provenance verification of the recovered frontend assets

Each asset was re-extracted **fresh from the signed binary** and hash-compared with the
preserved copy (this phase):

```js
// node verify-extract.js  (zlib.brotliDecompressSync over exe.subarray(offset))
// assets: index.html@30807145, index-COO2fUbz.js@28509689, index-CvNvudRa.css@31489343
```

| Asset | Offset in `digiexam.exe` | Decompressed size | Fresh SHA-256 == preserved SHA-256 |
|---|---|---|---|
| `index.html` | 30,807,145 | 1,167 B | ✅ MATCH |
| `assets/index-COO2fUbz.js` | 28,509,689 | 10,547,069 B | ✅ MATCH |
| `assets/index-CvNvudRa.css` | 31,489,343 | 144,919 B | ✅ MATCH |

Chain: signed binary (SHA-256 `047BA6…`) → byte offsets → brotli stream →
decompressed asset → SHA-256 identical to preserved copy. **The extraction is fully
reproducible.**

## 4. Prior-session working files (referenced, not committed)

Phase-1 analysis scripts remain in `%TEMP%\opencode\` (volatile); their methodology is
described above and in the phase-1 report. Key artifacts were copied into
`recovered-assets/` for preservation before temp cleanup could destroy them.
