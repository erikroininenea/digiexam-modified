# 09 — Windows integration & IPC

**Phase:** 2 (2026-11-16). Research agent C + review agent cross-check (all pipe offsets byte-verified by coordinator). Read-only; descriptive only — **no bypass content**. Labels: VERIFIED / STRONGLY INFERRED / UNVERIFIED / NOT APPLICABLE.

## 1. Windows service integration

| Item | Finding | Status |
|---|---|---|
| Service name | `DigiexamWindowsService` ("Digiexam Service") | VERIFIED (binary strings + live SCM) |
| Executable | `D:\Digiexam\service.exe` | VERIFIED (live `Win32_Service`) |
| State / start type | Running / **Automatic** | VERIFIED **from the live system** (not derivable from the binaries — no auto-start strings) |
| Registration | **No `CreateServiceW`/`DeleteServiceW` in any of the three binaries** (0 hits, review-verified) → registration/removal is performed by the **MSI installer** (uninstall entry `MsiExec.exe /X{00B4C41B-DAE8-4690-BC19-A5310E62D6DC}`, v26.4.7) | VERIFIED |
| Self-configuration | `ChangeServiceConfig2W` + "failed to configure service to restart on failure" → failure-recovery actions | VERIFIED string / STRONGLY INFERRED semantics (`SERVICE_CONFIG_FAILURE_ACTIONS`) |
| SCM API surface | service.exe: full dispatcher set (`StartServiceCtrlDispatcherW`, `RegisterServiceCtrlHandlerExW`, …); client: `OpenSCManagerW/OpenServiceW/StartServiceW/QueryServiceStatusEx/ControlService` | VERIFIED |
| Session targeting | `WTSEnumerateSessionsW`, `WTSQueryUserToken`, `WTSGetChildSessionId`, `LoadUserProfile`, `CreateEnvironmentBlock` — service locates the interactive session "running digiexam" and acts within it; same WTS strings also exist in `digiexam.exe` (review correction — not service-exclusive) | VERIFIED strings / usage STRONGLY INFERRED |

## 2. Named-pipe IPC (final consolidated matrix)

All five pipe literals byte-verified (raw file offsets):

| Pipe | `digiexam.exe` | `service.exe` | `toolbox-ipc-proxy.exe` | Role |
|---|---|---|---|---|
| `lockdown_windows_service_ipc` | 28200848 | 12981360 | — | main executor/control channel |
| `lockdown_notify_ipc` | 28201440 | 12981288 | — | service→client notify (abort) |
| `lockdown_lite_ipc` | 28199712 (+5 copies) | 12997456 | — | lockdown-lite plugin channel |
| `service_snapshot_collector_ipc` | 33455664, 33523568 | 13095420, 13248880, 13286912 | — | snapshot collection |
| `digiexam_toolbox_service_ipc` | 28200792 (+updater cluster 28502336) | **absent** | 750296 (+2 copies) | toolbox/update broker |

- Pipe prefix `\\.\pipe\` stored as a separate literal/template (`\\.\pipe\?`) —
  VERIFIED; runtime concatenation STRONGLY INFERRED.
- **Peer validation (both directions):** "unauthorized IPC client attempted to connect"
  (raw svc 13000872) / "client passed validation" (13001088) /
  "failed to validate process signature" + `lockdown-ipc\src\codesign.rs:106` — VERIFIED.
  Mechanism details intentionally not analyzed further.
- **Message vocabulary (serde, VERIFIED):** `ExecutorIpcRequest` = {Ping,
  RequestLockdownStart, RequestLockdownStartWithDefinitions, RequestLockdownStop,
  RollbackOperations, LockdownStopComplete, NotifyAbort}; `NotifyServerIpcRequest`
  carries control-adjacent variants; `CorrelatedInstallerRequest` {correlation_id, …}
  on the toolbox channel.

### 2.3 Toolbox topology — resolved as far as static evidence allows

- `digiexam.exe` (module `service\updater\updater_service.rs` @0x01E87050 cluster)
  **spawns** `toolbox-ipc-proxy.exe`, feeds requests via **stdin**, proxy forwards to
  the pipe and replies on **stdout** — VERIFIED (spawn/stdin/stdout/error strings both sides).
  **Correction of an earlier draft claim:** `service.exe` contains **neither**
  `updater_service` nor `toolbox-ipc-proxy` strings — the *client* runs the proxy.
- Who **listens** on `digiexam_toolbox_service_ipc`: **not service.exe** (literal absent;
  the fragmented literals there spell the *filename* `digiexamtoolboxservice.exe` —
  `olboxservice.exe` @13313760 + `digiexamtoolboxs` @13313776 — not a pipe name). The
  complete 26-char filename string exists in **no** binary, and no such executable
  exists on disk (`D:\Digiexam` = 3 exes; `C:\Program Files\Digiexam` is stale 26.1.24
  debris with no executables; SCM lists only `DigiexamWindowsService`).
- **Conclusion (review-resolved):** the elevated "toolbox service" role referenced by
  shared `lockdown-ipc` client code is **not among the shipped binaries** — either an
  unprovided component (`digiexamtoolboxservice.exe`, e.g. installed by a different SKU
  or downloaded) or `digiexam.exe` self-hosts (it holds the literal twice). **UNVERIFIED
  which; service.exe hosting refuted.**

## 3. Windows API families in use (descriptive, VERIFIED import strings)

| Family | Representative APIs |
|---|---|
| Process | `CreateProcessW`, `CreateProcessAsUserW`, `OpenProcess`, `TerminateProcess`, `CreateToolhelp32Snapshot`, `QueryFullProcessImageNameW`, psapi |
| Registry | `RegOpenKeyExW`, `RegCreateKey(Transacted)W`, `RegSetValueExW`, `RegDeleteValueW`, `RegDeleteTreeW` |
| Power | powrprof `Power{Create,Set,Clear}Request`, `PowerSetActiveScheme`, `RegisterPowerSettingNotification` |
| Keyboard/hooks | `SetWindowsHookExW`, `CallNextHookEx`, `UnhookWindowsHookEx` |
| WMI/COM | `IWbemLocator`/`IWbemClassObject`, WQL `SELECT * FROM ?` + `WITHIN ?` (event queries); classes incl. `Win32_BIOS`, `Win32_OperatingSystem`, `Win32_Processor`, `Win32_PhysicalMemory`, `Win32_VideoController`, `Win32_DiskDrive`, `Win32_NetworkAdapterConfiguration`, `Win32_ComputerSystem`, `Win32_DeviceGuard`, `Win32_Process`, `Win32_Service` |
| Code signing/trust | `WinVerifyTrustEx`, `WTHelper*` chain helpers, `CryptCATAdmin*` catalog hash APIs |
| Mitigations | `SetProcessMitigationPolicy` (+ success/failure log strings) |
| Pipes/IO | `NtCreateNamedPipeFile`, overlapped `ReadFile/WriteFile`, IO completion ports, `ConvertStringSecurityDescriptorToSecurityDescriptorW` (SDDL DACL construction) |
| Tokens/session | `OpenProcessToken`, `DuplicateTokenEx`, `AdjustTokenPrivileges`, WTS set (above) |
| Devices | `SetupDiGetClassDevsW`, `SetupDiEnumDeviceInfo`, … (device accessibility checks) |

Handler modules owning these (`handlers\windows\*: process_monitor, process_reviver,
hotkey_blocker, sleep_blocker, power_button_blocker, power_event_monitor, rdp_blocker,
task_blocker, update_blocker, service_{stopper,starter,status_checker,existence_verifier},
registry_*, vm_detector, audio_monitor, mac_address_checker, serial_checker, …`) are
catalogued descriptively in phase-1 §4 — no operational parameters or evasion content
is reproduced here.

## 4. Operation execution model (observable lifecycle)

- **Start:** config source selection ("fetching executor configuration" vs
  "client-supplied configuration"), idempotent replay acknowledgment; per-operation
  rollback entry created **before** execution ("Created rollback operation for ?") —
  STRONGLY INFERRED from pairing with `has_rollback`.
- **Results:** `ExecutionResult {operation_id, operation_name, status, executed_at}`
  with statuses Started/Stopped/Error/ActionRequired; notifications
  `OperationRequiresUserAction`, `ManualProcessKillingRequired` — pool-level VERIFIED,
  enum assignment STRONGLY INFERRED.
- **Watchdog:** 2× heartbeat interval = warning; **4× = presumed client death → abort +
  full rollback** — VERIFIED (extends phase-1).
- **Recovery log:** `service_recovery_log` (service) and `dx_recovery_log` (client side)
  — persisted before execution, validated/read at startup ("failed to rollback from
  existing recovery log"), cleared on clean stop — VERIFIED.
- **Privileged relay:** "relaying request to privileged client" — topology UNVERIFIED.

## 5. Open questions (UNVERIFIED)

- Host of `digiexam_toolbox_service_ipc` (§2.3).
- Exact SDDL string / pipe DACL; `ExecutionResult.status` enum membership; which
  component is the "privileged client"; start-type configuration path in the MSI.
