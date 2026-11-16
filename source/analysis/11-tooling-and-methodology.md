# 11 — Phase-2 tooling & methodology log

**Requirement:** document which skills, REA agents and MCP tools were *actually* used,
what each contributed, and which were not applicable. Everything below reflects real
calls made in this session (2026-11-16).

## 1. Installed skills — inspected

| Skill | Used | Contribution |
|---|---|---|
| `reverse-engineer-anything` (REA) | **Yes (instructions + routing)** | Routed the investigation; provided the CLI fallback rule (`npx -y rea-agents@6.9.0`) when the REA MCP server is absent; defined evidence discipline (observations vs inferences vs unknowns). Reference guides (`javascript-applications.md`, `native-and-artifacts.md`) informed methodology. |
| `browser`, `opencode`, `report` | Not used | No in-app-browser evidence, OpenCode-internals questions, or bug reports were involved. |

**REA MCP server status:** **NOT REGISTERED** in this environment (catalog search: no
`rea` namespace). Not fabricated, not assumed — verified by catalog inspection. The REA
*CLI fallback* was used in phase 1 (documented in
[`05-tooling-and-reproducibility.md`](05-tooling-and-reproducibility.md)); in phase 2 the
"REA agents" requirement (delegated independent research + review + cross-checking) was
implemented through the harness **subagent** tool (below), with REA's evidence-labeling
methodology applied to every report.

## 2. MCP servers and tools — actual usage

| Server / tool | Used | Contribution |
|---|---|---|
| **context7** `resolve-library-id` | **Yes** — 9 resolutions | Resolved official IDs: Tauri `/tauri-apps/tauri-docs`, plugins `/tauri-apps/plugins-workspace`, WebView2 `/websites/learn_microsoft_en-us_microsoft-edge_webview2`, Rust `/rust-lang/rust`, tonic `/websites/rs_tonic`, React `/reactjs/react.dev`, Redux Toolkit `/reduxjs/redux-toolkit`, Vite `/vitejs/vite`, gRPC `/grpc/grpc.io`, protobuf `/protocolbuffers/protocolbuffers.github.io` |
| **context7** `query-docs` | **Yes** — 8 queries | Verified official behavior: Tauri v2 IPC (custom-protocol, JSON-RPC-like commands, event/Channel model); Tauri v2 capabilities/ACL (allow/deny grammar, default-allow custom commands, `AppManifest::commands`); WebView2 Evergreen distribution + process model; updater plugin Windows exit-after-install contract; tonic `/package.Service/Method` path convention; RTK `createAsyncThunk`/`createSlice` patterns; Vite `assets/[name]-[hash].[ext]` naming; gRPC-over-HTTP/2 path rules. **Separated official framework behavior from DigiExam-specific implementation in docs 07/08/10.** |
| **dnspy-mcp** | **Yes — tested, NOT APPLICABLE (proven live)** | `list_assemblies` → `[]`; `open_files("digiexam.exe")` → `failed_count: 1`, `"could not load (not a .NET assembly/module?)"`. dnSpy *itself* rejects the native Rust binary; consistent with `BSJB` absence. Documented, not forced. |
| **github** | **Yes** | `search_repositories` (repo identity/state), plus git CLI for inspect/commit/push. |
| **blender-mcp** | Not used | 3D-scene tooling; no relevance. NOT APPLICABLE. |
| **opencode** (session/worktree/MCP-resource tools) | Not needed | No session moves or MCP resources required. |

## 3. Subagent delegation (the four required research tracks)

All agents worked **read-only** under explicit constraints (no execution, no patching,
no bypass documentation, quote limits, mandatory VERIFIED/STRONGLY INFERRED/UNVERIFIED
labels + contradictions section). Reports preserved in `%TEMP%\opencode\phase2\{A,B,C,D}.md`.

| Agent (session) | Track | Key contributions |
|---|---|---|
| A (`ses_e1ad465faffe…`) | Rust/Tauri architecture | Cargo workspace map; 188/130/10 crate-version inventories; dual reqwest/tonic; TLS stacks; `optl` telemetry layer; build-provenance anomaly (different builder for the proxy); shared lockdown-core in client+service |
| B (`ses_e1ad465f8ffe…`) | React/Redux/Vite frontend | Pinned React 18.3.1, RTK 2.x, Router 6, MUI/i18next/connect-es; 19 slices; custom middleware; 199-RPC API surface; PLATFORM folding; found the 26.4.4 version anomaly |
| C (`ses_e1ad465f5ffe…`) | Windows integration/IPC | Pinned all 5 pipe offsets; SCM/WMI/WinVerifyTrust/WTS/mitigation API families; executor/rollback/recovery-log lifecycle; watchdog 2×/4× stages; CreateServiceW absence → installer-based registration |
| D (`ses_e1ad465f2ffe…`) | WebView2/lifecycle | wry git-checkout; webview-manager command surface + injected restriction script (descriptive); deep-link/startup/update/shutdown sequences; CrabNebula endpoint; Evergreen loader evidence; h2 false-friend excluded |
| Review (`ses_e1acc0d19ffe…`) | Cross-check (agents "share findings and resolve contradictions") | 11-item contradiction checklist resolved with byte-level evidence; 8 corrections (pipe host refuted for service.exe; updater_service is client-only; wry git checkout; prost upgrade; minisign base64 phrasing; WTS not service-exclusive; fragmented `digiexamtoolboxservice.exe` filename; start_ping absent on Windows) |

## 4. Verification performed by the coordinator (not delegated)

- Re-hashing/re-extraction chain (phase 2 start) — byte-exact matches.
- Live SCM/registry inspection: service Running/Automatic; MSI uninstall entry.
- Bundle version anomaly: exactly one `26.4.4`, zero `26.4.7`.
- Pipe-name offset verification: 4/4 byte-exact.
- Tool-inventory inspection before any research claim (this file, §1–2).

## 5. Tooling lessons (for future phases)

- **Dir-wide grep over the strings dumps under-reports** (review agent hit false
  "no match" several times) — use per-file grep plus raw byte scans for binary literals.
- PowerShell 5.1: no `??` operator; ScriptBlock `.Invoke(...).Count` returns the wrapper
  count, not the match count (phase-2 marker-scan pitfall, see
  [`05-tooling-and-reproducibility.md`](05-tooling-and-reproducibility.md)).
- Node `toString('latin1')` returns a *string* — use `substring`, not `subarray`.
- The 10.5 MB minified bundle defeats whole-file JS analyzers (phase-1 REA CLI runs);
  targeted regex-with-offset scans are the effective tool at this size.

## 6. Methodology status against the phase-2 plan

1. Inspect installed skills — **done** (§1)
2. Inspect MCP servers/tools — **done** (§2)
3. Read skill instructions — **done** (REA skill loaded in phase 1, applied throughout)
4. Context7 documentation — **done** (§2; incorporated into docs 07/08/10)
5. REA delegation — **done** via 4+1 subagents (§3)
6. Verify against binary metadata — **done** (coordinator + review agent)
7. Consolidate — **done** (contradiction log closed; corrections applied)
8. Update documentation — **done** (docs 06–11 + README index)
