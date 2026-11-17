# 03 — Component architecture & interfaces

**Scope note:** this document describes architecture, interfaces and *observable*
behavior as recorded by read-only static analysis. It intentionally does **not**
document any technique for weakening or bypassing the lockdown — see
[`../documentation/legal-and-scope.md`](../../documentation/legal-and-scope.md).
Fine-grained detail (operation catalog, registry values, hook mechanisms) is preserved
in the phase-1 report: [`../documentation/digiexam_lockdown_rapport.md`](../../documentation/digiexam_lockdown_rapport.md).

## 1. Component map

```
DigiExam backend (*.digiexam.com)
   │  gRPC over TLS, OAuth v2 token endpoint
   │  lockdown.service.v1.LockdownService/GetLockdownConfig
   │  assignment.service.v1.AssignmentService/SetConnectivityStatus
   ▼
digiexam.exe  ── Tauri 2.10.3 host (Rust core)
   │   ├─ WebView2 webview: React/Redux frontend bundle
   │   ├─ Tauri plugins: lockdown-lite, webview-manager, io, battery,
   │   │   keyboard-layout, store, updater, deep-link
   │   └─ starts toolbox-ipc-proxy.exe via stdin pipe
   │
   ├── named pipe  lockdown_windows_service_ipc   (config + session → service)
   ├── named pipe  lockdown_notify_ipc            (client → service abort channel)
   ├── named pipe  lockdown_lite_ipc              (client-internal "lite" lockdown path)
   ├── named pipe  service_snapshot_collector_ipc (snapshot collection)
   └── named pipe  digiexam_toolbox_service_ipc   (toolbox proxy broker)
   ▼
service.exe  ── "DigiexamWindowsService" (Rust, runs as service)
   ├─ operation executor: start / stop / rollback operations
   ├─ recovery log on disk (rollback after crash)
   ├─ watchdog: client heartbeat; abort + rollback after 4× heartbeat interval
   └─ legacy flow: service itself can fetch GetLockdownConfig over gRPC
   ▼
toolbox-ipc-proxy.exe  ── IPC broker ("digiexam_toolbox_service_ipc"),
                          started by the client, stdin-pipe handshake
```

## 2. gRPC / protobuf surface (observed service strings)

- `lockdown.service.v1.LockdownService`:
  `GetLockdownConfig`, `LockdownStart`, `LockdownStartWithDefinitions`,
  `LockdownStop`, `RollbackOperations`
- `assignment.service.v1.AssignmentService/SetConnectivityStatus`
  (examId/studentId connectivity reporting)
- definition/parameter schemas: `lockdown.opdefinition.v1`,
  `lockdown.lifecycle.v1`, `lockdown.parameter.v1`

**Observation:** lockdown configuration is **server-driven**: the backend supplies
`OperationDefinition` lists filtered by OS id, architecture, version range
(`min/max`), `require_administrator`, and validity window (`valid_from`/`valid_to`).
The client forwards the configuration plus `SessionInfo`
(`organization_id`, `user_id`, `exam_id`, `is_proctored`, tokens,
`client_session_id`) to the service over the named pipe. The binaries also carry a
built-in **default operations catalog** (phase-1 report §4) used when definitions are
embedded client-side.

## 3. Lockdown lifecycle (as observable from log strings)

1. **Start** — client (or legacy service flow) obtains configuration; service logs
   `"initializing lockdown from client-supplied configuration"`; replayed requests are
   acknowledged idempotently (`"Lockdown already running"`).
2. **Execution** — each operation is started, journaled into the on-disk **recovery
   log**, and reports `ExecutionResult` states (`OperationStarted`, `OperationRolledBack`,
   `Error`, `ActionRequired`; the latter two surface in the UI as `action_required_error`).
3. **Stop/crash safety** — stop requests produce per-operation rollback entries
   (`"Created rollback operation for ?"`) executed in reverse order; the service-side
   watchdog aborts and rolls back if the client misses
   *"4 times the heartbeat interval"*; a dead service is covered by the client notify
   channel plus the persistent recovery log.

## 4. Operation categories (summary — full catalog in phase-1 §4)

Descriptive category list only; parameters and concrete registry values are in the
phase-1 report:

| Category | Examples (observed handler names) |
|---|---|
| Input/system blockers | `WindowsHotkeyBlocker` (+ low-level keyboard hook), `SleepBlocker`, `PowerButtonBlocker`, `TaskBlocker`, `UpdateBlocker`, `RdpBlocker` |
| Service control | `ServiceStopper`, `ServiceStarter`, `ServiceStatusChecker` |
| Registry/policy setters & verifiers | `RegistryIntegerValueSetter`, `RegistrySetStringValue`, `RegistryValueDeleter`, `RegistrySubKeyDeleter`, `Registry*Verifier`, `FileExistenceVerifier`, `DllExportChecker`, `DeviceAccessibilityChecker` |
| Process supervision (anti-cheat, system level) | `ProcessMonitor` (Authenticode-signer whitelist + unsigned-hash whitelist, `kill_tree`), `ProcessReviver`, `SnapshotCollector` (WMI hardware/software/network inventory incl. `hypervisor`, MAC, IP) |
| Environment detectors (report-only) | `VmDetector`, `MacAddressChecker`, `SerialChecker`, `WindowsAudioMonitor`, `PowerEventMonitor` |
| Webview/UI level (client) | fullscreen enforcement, clipboard monitor/blocker, `set_content_protected`, `clear_all_browsing_data` |
| Service self-protection | process mitigation policies at service start (image-load + extension-point policies) |

## 5. IPC trust model (verified strings)

- The **service** validates connecting processes: `"unauthorized IPC client attempted
  to connect"` / `"client passed validation"`.
- The **client's** IPC server performs the same validation — the trust model is
  bidirectional.
- The client sends `session ID` + region (`set_server_uri`) to the service at startup.

**Hypotheses about the strength of this model (H1–H4) are recorded in phase-1 §8.5**
and deliberately left unverified: testing them would require an isolated,
vendor-authorized lab, and they are *not* reproduced here.

## 6. Electron↔native interface answer

The research plan asked for "interfaces between Electron, Node.js and native Windows
components". The factual answer after analysis:

- **No Electron/Node layer exists.** The equivalent interfaces are:
  1. **WebView2 ↔ Rust core:** Tauri IPC (invoke commands + events) restricted by the
     Tauri v2 capability/ACL system → [`04-frontend-ipc-surface.md`](04-frontend-ipc-surface.md)
  2. **Rust core ↔ OS service:** named pipes (`lockdown_*`), protobuf messages,
     process-PID-based peer validation
  3. **Service ↔ Windows:** Win32 (registry, services, scheduled tasks, WMI,
     `SetWindowsHookEx`, `SetProcessMitigationPolicy`, power events)
  4. **Client/service ↔ backend:** gRPC/tonic over TLS to `*.digiexam.com`
     (incl. `auth.`/`telemetry.` subdomains, `oauth/v2/token`)
