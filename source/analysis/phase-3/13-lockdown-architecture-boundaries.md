# 13 — Lockdown architecture as a boundary model

**Target:** DigiExam for Windows v26.4.7 (`digiexam.exe`, `service.exe`, `toolbox-ipc-proxy.exe`).
**Method:** read-only static analysis. This document synthesizes the phase-1 report
([`../documentation/digiexam_lockdown_rapport.md`](../../documentation/digiexam_lockdown_rapport.md), Swedish)
and phase-2 docs 03/04/06/07/09/10 into a single **boundary model**, extended with new
string-dump observations from this pass (cited as `strings_*.txt:line`). No binary was
executed or modified. **Descriptive architecture only — no bypass, weakening or
enforcement-parameter content.** Registry values, hotkey lists and whitelist mechanics
live only in phase-1 §4 and are *referenced*, never reproduced here.

**Labels:** VERIFIED (byte-observed in the dumps or byte-verified in a cited doc) ·
STRONGLY INFERRED · UNVERIFIED.

---

## 1. Process boundary map

### 1.1 Processes and privilege levels

| Process | Role | Privilege level | Status |
|---|---|---|---|
| `digiexam.exe` | Tauri 2.10.3 host: Rust core (`native\apps\digiexam`), WebView2 hosting, gRPC client, updater, `lockdown-lite` plugin, client-side recovery log (`dx_recovery_log`), spawns the toolbox proxy | Interactive user session, unprivileged. Both binaries contain token-elevation introspection strings ("failed to query elevation info on process token" analogues, `strings_digiexam.txt:421311`, `strings_service.txt:151852`); operation definitions carry a require-administrator flag | Elevation introspection VERIFIED; the client's own privilege level STRONGLY INFERRED from install/session context |
| **WebView2 runtime process group** (`msedgewebview2.exe` child set: browser / GPU / renderers) | Renders the exam UI and *remote exam content* (frontend bundle `index-COO2fUbz.js`) | WebView2's own multi-process model (public runtime behavior); no lockdown authority of its own | WebView2 backend VERIFIED (wry git-checkout `webview2\mod.rs` paths); child-process model STRONGLY INFERRED (public WebView2 architecture); renderer sandbox level UNVERIFIED |
| `service.exe` | Windows service **`DigiexamWindowsService`** (`native\apps\lockdown-windows-service`): operation executor, watchdog owner, recovery log (`service_recovery_log`), legacy gRPC config fetch | **LocalSystem expected.** An error path in *both* binaries states that the user-token query requires the service to be running as LocalSystem (`strings_service.txt:151795`, `strings_digiexam.txt:421253`); the service acts *into* the interactive session via WTS token/session APIs and user-token process creation (`CreateProcessAsUserW` failure strings in both binaries) | LocalSystem expectation VERIFIED (string); live token check of the running service UNVERIFIED in this pass (live SCM data was verified in doc 09 §1) |
| `toolbox-ipc-proxy.exe` | Thin stdio↔named-pipe forwarder (only `lockdown-ipc` linked; no SCM/advapi32 — doc 06 §3); child of `digiexam.exe` (spawned by the updater-service module, stdin handshake, stdout replies — doc 09 §2.3) | Inherits the spawning client's context (child process) | Topology VERIFIED (doc 09); inherited privilege STRONGLY INFERRED |
| `digiexamtoolboxservice.exe` (**unshipped role**) | Elevated "toolbox service" implied by the pipe name `digiexam_toolbox_service_ipc` and by installer-broker vocabulary present in the proxy (below) | Elevated (privileged installer broker) | **UNVERIFIED** — not among the shipped binaries; `service.exe` hosting refuted (doc 09 §2.3) |

### 1.2 Boundary crossings

```
                ┌──────────────────────────────────────────────┐
                │ DigiExam backend (*.digiexam.com, gRPC/TLS)  │
                └──────▲───────────────────────────▲───────────┘
        GetLockdownConfig (normal: client / legacy: service)
                       │                           │
  ┌────────────────────┴───────────────┐           │
  │ digiexam.exe  (user session)       │           │
  │  ├─ WebView2 process group         │           │
  │  │    └─ Tauri IPC (ACL-gated) ──┐ │           │
  │  ├─ lockdown-lite plugin ◄───────┘ │           │
  │  ├─ updater service module ──stdin/stdout──► toolbox-ipc-proxy.exe
  │  ├─ notify server ◄──?             │                    │ pipe
  │  └─ watchdog heartbeat sender ──┐  │                    ▼
  └─────────────────────────────────┼──┘          digiexam_toolbox_service_ipc
   lockdown_windows_service_ipc ────┼────►  ┌──────────────────────────┐
   lockdown_lite_ipc (client-lite)  │       │ (unshipped) toolbox      │
   service_snapshot_collector_ipc   │       │ service role             │
                                    ▼       └──────────────────────────┘
                      ┌───────────────────────────────────┐
                      │ service.exe = DigiexamWindowsService
                      │  (LocalSystem, SCM lifecycle)
                      │  ├─ executor + recovery log (disk)
                      │  ├─ watchdog (client heartbeat monitor)
                      │  ├─ WTS/token crossing → user session
                      │  └─ legacy gRPC fetch (GetLockdownConfig)
                      └───────────────────────────────────┘
```

| Crossing | Peers | Transport | Vocabulary (observed serde/protobuf names) | Status |
|---|---|---|---|---|
| **Tauri IPC** | WebView2 frontend ↔ Rust core | in-process custom-protocol (`ipc.localhost`), JSON | commands/events per doc 04/07 | VERIFIED |
| `lockdown_windows_service_ipc` | client ↔ service | named pipe, `lockdown-ipc` framing | `ExecutorIpcRequest` {Ping, RequestLockdownStart(WithDefinitions), RequestLockdownStop, RollbackOperations, LockdownStopComplete, NotifyAbort}; `SessionInfo` | VERIFIED (doc 09 §2 + dumps) |
| `lockdown_notify_ipc` | service ↔ client-side *notify server* | named pipe | `NotifyServerIpcRequest` (shares vocabulary — a RequestLockdownStart variant exists, `strings_digiexam.txt:269224`); abort strings both ends | Pipe + vocabulary VERIFIED; **server placement/direction UNVERIFIED** (§5) |
| `lockdown_lite_ipc` | client-internal "lite" path | named pipe | lite plugin commands (`run_all/stop_all/rollback/query_windows_service_status`) | VERIFIED literals (doc 09 §2) |
| `service_snapshot_collector_ipc` | SnapshotCollector operation ↔ collector server | named pipe | snapshot schema {`schema_version, collector_version, collected_at, platform, hardware, software, network, virtualization, os_architecture, …`} | Strings VERIFIED; service-side server module `lifecycle\snapshot_collector.rs` VERIFIED (`strings_service.txt:146396`); exact data direction STRONGLY INFERRED |
| `digiexam_toolbox_service_ipc` | client ↔ proxy (stdio bridge) ↔ toolbox service role | named pipe + stdin/stdout | `UpdateFromClient` {ClientStarted, RegionChanged(new_region), SessionChanged}; `InstallerRequest` {CheckVersion, Update, Uninstall, RunDigiexam, GetAppInfo}; `CorrelatedInstallerRequest` {correlation_id}; `AuthenticatedUpdateMeta`; plus lifecycle words (`LockdownStarting`, `RunOperation`, `StopOperation`, `RollbackOperations`, `Notify`, `Abort`) — `strings_toolbox.txt:12662–12801` | Strings VERIFIED; **listener UNVERIFIED** |
| **SCM** | service ↔ Service Control Manager; client queries/controls the service | SCM APIs | service-side dispatcher + status-change strings (`strings_service.txt:147138`); client-side status query incl. service-name string (`strings_digiexam.txt:415783`) | VERIFIED (doc 09 §1 + dumps) |
| **gRPC** | client ↔ backend; service ↔ backend (legacy) | tonic/h2 over TLS | `lockdown.service.v1.*`, `assignment.service.v1.*` incl. `CreateLockdownSystemSnapshotUpload` → response {`snapshot_id`, `upload_url`} (`strings_digiexam.txt:416083`) | VERIFIED (doc 10 + dumps) |
| **WTS/token crossing** | service → interactive user session | `WTSEnumerateSessionsW`, `WTSQueryUserToken`, `LoadUserProfile`, `CreateEnvironmentBlock`, `CreateProcessAsUserW` | `lockdown-windows\src\user.rs` panic paths present in the *client* too (`strings_digiexam.txt:421293`) — shared crate | Strings VERIFIED; usage STRONGLY INFERRED (doc 09 §1) |
| **stdio bridge** | client → proxy | stdin/stdout pipes | proxied request/reply | VERIFIED (doc 09 §2.3) |

**Privileged-relay vocabulary (new, both binaries):** strings describe *relaying a request
to a privileged client*, the absence of such a client, *a privileged service failing to
handle a request*, and *failure to connect to a privileged service*
(`strings_digiexam.txt:267636-267637, 412010`; `strings_service.txt:146202-146203, 146260`).
So the architecture contains a **relay concept** — some requests are forwarded to a
privileged counterpart — but the role assignment (which process is the "privileged
client", which the "privileged service") remains open (doc 09 §4). UNVERIFIED.

---

## 2. Trust boundary inventory

Each row: a trust boundary, the crossing mechanism, and the validation documented so far.

| # | Boundary | Validation / mechanism | Status |
|---|---|---|---|
| T1 | **Backend → client** (lockdown-config provenance) | gRPC over TLS; OAuth token flow; configuration is **server-driven**: `OperationDefinition` lists filtered by platform, arch, version range, validity window and a require-administrator flag; client forwards it + `SessionInfo` to the service (doc 03 §2, phase-1 §1) | VERIFIED; *strength of provenance validation at the service* = phase-1 H1, deliberately untested |
| T2 | **Updater / update artifacts** | tauri-plugin-updater + `minisign-verify 0.2.5`; embedded minisign public key (key-ID published in doc 10 §4); installer path (`msiexec`) present **only in `digiexam.exe`** (`strings_digiexam.txt:418788-418789`); Windows contract = verify → install → exit (doc 07 §5). New: toolbox-channel update metadata is a struct pairing the update payload with a **token** (`AuthenticatedUpdateMeta{updateMeta, token}`), wrapped in `CorrelatedInstallerRequest{correlation_id}` (`strings_digiexam.txt:269077, 269081`; `strings_toolbox.txt:12662`) | VERIFIED strings; runtime verification behavior matches the official plugin contract per doc 07 |
| T3 | **WebView2 frontend → Rust core** | Tauri v2 capability/ACL system (see §3) | VERIFIED (table); runtime grants UNVERIFIED |
| T4 | **Client ↔ service named pipes** | **Bidirectional peer validation** via `lockdown-ipc\src\codesign.rs`: both the service and the client's own IPC server reject unvalidated peers ("unauthorized IPC client attempted to connect" / "client passed validation" — phase-1 §8.4, doc 09 §2; anchors `strings_service.txt:151348-151352`). Pipe protection is built through a **SDDL security-descriptor conversion** API; an invalid-descriptor error string exists in **all three** binaries (`strings_digiexam.txt:421150`, `strings_service.txt:151326`, `strings_toolbox.txt:12815`) | Peer-validation strings + descriptor error path VERIFIED; **exact DACL/SDDL content UNVERIFIED** (doc 09 §5) |
| T5 | **Session binding** | Requests carry `SessionInfo` — serde fields confirm `organization_id, user_id, exam_id, is_proctored` plus token fields and `client_session_id` (`strings_service.txt:146668`); the client notifies the service of the session ID at startup and propagates region/session changes (`UpdateFromClient::{ClientStarted, RegionChanged, SessionChanged}` — `strings_toolbox.txt:12666`); the service re-targets its logger when the session changes (`strings_service.txt:146620`). New: **handlers are session-bound** — both binaries contain error paths stating session info must be set for a handler/snapshot collector (`strings_digiexam.txt:416141`, `strings_service.txt:147509`), i.e. operations refuse to run unbound | VERIFIED |
| T6 | **Client liveness (watchdog)** | Client sends heartbeats to the service-owned watchdog; twice the heartbeat interval → warning; four times → client presumed dead → abort + full rollback ("client failed to respond within 4 times the heartbeat interval…" — doc 09 §4, phase-1 §3). Mirrored client-side strings ("lost connection with watchdog") trigger an abort request through the notify server (phase-1 §3) | VERIFIED (published) |
| T7 | **Abort channel** | `NotifyAbort` variant on the executor vocabulary; abort-request failure strings on both ends; the notify server tracks operation state (a new string indicates it can be "unaware of operations stopping", `strings_digiexam.txt:416613`) | Vocabulary VERIFIED; topology UNVERIFIED |
| T8 | **Process trust inside operations** | WinVerifyTrust / catalog-hash import surface (doc 09 §3); operation mechanics intentionally not reproduced (phase-1 §4) | VERIFIED imports |
| T9 | **Client/service self-protection** | Process mitigation policies — **image-load and extension-point policies with success/failure log strings exist in *both* binaries** (`strings_digiexam.txt:441550-441551`, `strings_service.txt:147261-147262`) — plus the **`anti-dll-injection` crate also compiled into both** (panic/log anchors in both dumps; doc 06 §1 listed it as client-only — *extension, not contradiction*: the shared crate is linked in both hosts) | VERIFIED |
| T10 | **Elevation gating of operations** | Operation definitions carry `require_administrator` (serde aliases observed in **all three** binaries incl. the proxy, `strings_toolbox.txt:12752`); both main binaries query their own token elevation | VERIFIED strings; which host executes admin-requiring ops STRONGLY INFERRED = the LocalSystem service executor |
| T11 | **Snapshot egress** | Collection is session-bound (exam/student IDs in the handler's event context, `strings_digiexam.txt:416162`); results leave only via the backend upload RPC returning an upload URL (T1/gRPC row) | VERIFIED structure; actual network use not observed (static) |
| T12 | **WebView2 as content trust boundary** | Remote exam content and backend JS live behind the webview boundary with **no direct OS authority**: every privileged action requires Tauri IPC → first-party plugin → validated pipe → service. Native-side webview hardening exists (`set_content_protected`, `clear_all_browsing_data` — phase-1 §4, doc 04 §4) | Architecture VERIFIED; renderer sandbox level UNVERIFIED |

---

## 3. Capability boundary between webview and native (Tauri ACL)

Established in phase-1 §8.2 and docs 04 §4 / 07 §3–4; summarized here, not re-copied:

- `digiexam.exe` embeds a **Tauri v2 capability/ACL table** (offset region ≈ 33,070,000–33,076,000;
  `allow-<kebab>`/`deny-<kebab>` identifiers plus snake_case command names — direct allow/deny
  pairs for lockdown-lite commands visible at `strings_digiexam.txt:414635`). VERIFIED.
- The ACL binds **which commands the webview may invoke per plugin**: core window/webview
  commands, `lockdown-lite` (`run_all`, `stop_all`, `rollback`, `query_windows_service_status`,
  clipboard set/clear, permission dialogs), `io`, `battery`, `keyboard-layout`, `updater`
  (`download_and_install` + `exe`-extension allow; bundle-type deny). Explicit **deny**
  entries in specific scopes form a least-privilege tightening pattern. Full table: doc 04 §4.
- **Custom app commands** (exam connectivity, timers, metadata/logger, `rollback_from_recovery_log`,
  `set_server_uri`, …) are registered separately and, per official Tauri v2 semantics
  (Context7-verified, doc 07 §3), are allowed unless restricted at build time.
- `internal_toggle_devtools` appears in the allow-list — an ACL-level devtools surface;
  **runtime exposure UNVERIFIED** (phase-1 H4).
- **The decisive architectural point:** the webview has *no pipe, no SCM and no gRPC authority
  of its own*. Its whole native reach is the ACL-gated invoke surface, and the only path toward
  the enforcement engine is `plugin:lockdown-lite|…` → `lockdown-ipc` client → codesign-validated
  pipe (T4). Pipe peer validation is **process-signature-based on the client process**, so the
  gate sits at the process boundary, not inside the webview (phase-1 §8.4, doc 03 §5).
  Static ACL presence ≠ runtime grant state — which commands the *shipped webview
  configuration* can actually exercise is UNVERIFIED (doc 07 §6).

---

## 4. Lifecycle states

State names below are analytical labels; anchors are published strings. No parameter values.

1. **IDLE** — service running under SCM (failure-recovery actions configured; "service marked
   as running"), no lockdown active; client can query service status at any time.
2. **CONFIG_ACQUISITION** — normal flow: client fetches `GetLockdownConfig`; legacy flow:
   the service itself fetches executor configuration (published string). Replayed start
   requests are acknowledged idempotently ("Lockdown already running").
3. **STARTING** — executor receives the client-supplied configuration + `SessionInfo`
   (published string); a **rollback entry is created per operation before execution**
   (doc 09 §4, STRONGLY INFERRED from the pairing); handlers must be session-bound (T5).
4. **RUNNING** — operations started and **journaled** into the on-disk recovery log
   (registration is a distinct, fallible step per error strings in both binaries); the
   heartbeat loop is active; per-operation `ExecutionResult`s (Started/Stopped/Error/
   ActionRequired) propagate to the client UI (`lockdown_started`, `critical_lockdown_error`,
   `action_required_error`, `abort` — doc 04 §2).
5. **STOPPING** — stop requested; per-operation stop + rollback creation (published string);
   observed edge cases: stopping an operation that isn't running; the notify server being
   unaware of operations stopping (T7).
6. **ABORT (watchdog path)** — heartbeat silence ≥ 4× interval → client presumed dead →
   abort + full rollback by the service (T6). Complement: client loses the watchdog →
   abort request via the notify server (T7).
7. **ROLLED_BACK / CLEAN** — rollback entries executed (reverse order per phase-1); the
   recovery log is cleared (its failure path is a published string); `LockdownStopComplete`
   closes the exchange.
8. **CRASH_RECOVERY** — on (re)start with a leftover log: read → validate → roll back
   (published failure paths in both binaries); recovery-log actions are performed as discrete
   steps (new string, both binaries); an unknown handler for a logged rollback is a distinct
   failure state (published string). Two persistent logs exist by host: `service_recovery_log`
   (service) and `dx_recovery_log` (client) — doc 09 §4.
9. **CLIENT-LITE PATH** — a parallel, client-internal state set over `lockdown_lite_ipc`
   with its own client-side recovery log and a `rollback_from_recovery_log` command; used
   when lockdown is disabled (open-book exams), per doc 04 §3.
10. **SERVICE-LEVEL (SCM) STATES** — the service's own SCM status machine plus
    failure-recovery actions ("failed to configure service to restart on failure" — doc 09 §1).

---

## 5. Open boundary questions (UNVERIFIED only)

1. **Toolbox pipe listener / `digiexamtoolboxservice.exe`** — still unresolved (doc 09 §2.3).
   New proxy vocabulary (installer/version broker messages, operation-level words) refines the
   *inferred role* but does not identify the host.
2. **"Privileged client" / "privileged service" relay topology** — which shipped (or unshipped)
   component fills each role; both main binaries carry the relay strings (§1.2).
3. **Notify-channel direction** — where the notify server actually listens (client-side vs
   service-side) and the exact abort propagation path; doc 09's matrix records the
   service→client reading, the shared string pool keeps it ambiguous.
4. **Runtime-effective ACL grants** and whether the child exam webview (if any) holds a
   different capability set than the main window (doc 07 §6).
5. **Pipe DACL specifics** — SDDL construction confirmed in all three binaries, content not
   (doc 09 §5).
6. **Phase-1 hypotheses H1–H4** (config-provenance validation strength, watchdog margining,
   rollback closedness under power loss, devtools exposure) — referenced, not re-tested;
   verifying them requires a vendor-authorized lab (phase-1 §8.5).
7. **WebView2 renderer sandbox level** for exam content, and whether any privileged operation
   is reachable without the pipe hop (no such path observed statically).
8. **optl telemetry boundary** (encrypted local telemetry, endpoints unknown — doc 06 §6) and
   the **region→host map** (doc 10 §6).
9. **Lite vs. full executor distribution** per exam type — server-defined, only observable in
   a live authorized session (doc 04 §6).
10. **LocalSystem assumption** — the LocalSystem expectation string appears at user-token
    query time; whether this is an enforced check or a diagnostic assumption is UNVERIFIED.

---

### Extensions/corrections recorded by this pass

- `anti-dll-injection` crate: present in **both** binaries (doc 06 listed client-only) — shared
  self-protection across hosts.
- Image-load/extension-point mitigation-policy log strings: present in **both** binaries
  (phase-1 located them at service start only).
- Service privilege expectation (LocalSystem) is now string-VERIFIED, previously only implied
  by the service role.
- SnapshotCollector: service-side server module under `lifecycle\` + session-bound handler
  requirement + backend upload RPC response shape — the egress boundary (T11) is now mapped.
- Toolbox channel: fuller message vocabulary (`UpdateFromClient`, `InstallerRequest`,
  `CorrelatedInstallerRequest`, `AuthenticatedUpdateMeta`, lifecycle words) — the proxy is a
  forwarder for authenticated installer/version traffic and operation-adjacent messages.
