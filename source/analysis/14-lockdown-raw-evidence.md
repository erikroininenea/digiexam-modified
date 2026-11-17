# 14 — Lockdown component locations & raw string evidence

**Phase:** 3 (2026-11-16). **Purpose:** publish the *exact locations* (binary + raw file
offset) and *raw string evidence* for every lockdown-related component identified in
docs 06/13 — the "where", not the "how". **Scope note:** the shipped binaries contain
no recoverable source code; the only text-form artifacts are embedded strings and
compiled machine code. This document publishes the string evidence with byte-exact
offsets. Machine-code dumps and any enforcement internals (registry values, hotkey
lists, whitelist contents) are intentionally **not** reproduced — see
[`../documentation/legal-and-scope.md`](../documentation/legal-and-scope.md) and
[`13-lockdown-architecture-boundaries.md`](13-lockdown-architecture-boundaries.md).

**Labels:** every offset below is **VERIFIED** (byte-exact literal match at the cited
raw file offset, re-verified 2026-11-16 against the SHA-256-hashed binaries in
[`../recovered-assets/MANIFEST.md`](../recovered-assets/MANIFEST.md)). Offsets are
decimal-file offsets in the shipped binaries (not RVA/VA; they are version-specific
to 26.4.7 and will differ in other builds).

**Method:** raw Latin-1 byte search over the full binaries (script in
[`../extraction/TOOLING-SNIPPETS.md`](../extraction/TOOLING-SNIPPETS.md) §8). Where a
literal occurs many times (Rust panic-location metadata repeats per call site), the
table shows the first offset + occurrence count.

---

## 1. `digiexam.exe` (client) — lockdown components

### 1.1 App shell & command layer (`native\apps\digiexam\src`)

| Module | First offset | Occurrences | Raw literal |
|---|---|---|---|
| `main.rs` | `0x01B23063` | 2 | `native\apps\digiexam\src\main.rs` |
| `lib.rs` | `0x01AE7E22` | 17 | `native\apps\digiexam\src\lib.rs` |
| `command\exam.rs` | `0x01AE8988` | 19 | `native\apps\digiexam\src\command\exam.rs` |
| `command\sec.rs` | `0x01AE9790` | 1 | `native\apps\digiexam\src\command\sec.rs` |
| `service\optl.rs` | `0x01E75F40` | 2 | `native\apps\digiexam\src\service\optl.rs` |
| (client rollback entry) | `0x01AE7E08` | 1 | `rollback_from_recovery_log` |
| (client recovery log name) | `0x01B2E8FE` | 2 | `dx_recovery_log` (also `0x01FD9FE8`) |

### 1.2 First-party lockdown plugin (`tauri-plugin-lockdown-lite`)

| Module | Offset | Raw literal |
|---|---|---|
| `src\commands.rs` | `0x01AE55AB` | `tauri-plugin-lockdown-lite\src\commands.rs` |
| `src\handler.rs` | `0x01AE6C5C` | `tauri-plugin-lockdown-lite\src\handler.rs` |
| `src\lib.rs` | `0x01AE7142` | `tauri-plugin-lockdown-lite\src\lib.rs` |
| `src\setup_component\windows\clipboard_monitor.rs` | `0x01B12AC3` | `tauri-plugin-lockdown-lite\src\setup_component\windows\clipboard_monitor.rs` |

Bare plugin name `tauri-plugin-lockdown-lite`: 44 occurrences (first `0x01AE55AB`) —
command-registry and ACL references. Frontend anchor for the plugin command surface:
the literal `plugin:lockdown-lite|run_all` sits at **char offset 5,618,412** in the
recovered bundle `index-COO2fUbz.js` (see doc 08 for the bundle's provenance).

### 1.3 Shared lockdown crates linked into the client

| Crate module | First offset | n | Raw literal |
|---|---|---|---|
| `lockdown-core\src\handler\mod.rs` | `0x01AE5DF4` | 65 | `lockdown-core\src\handler\mod.rs` |
| `lockdown-core\src\handler_factory.rs` | `0x01FC8C6F` | 1 | `lockdown-core\src\handler_factory.rs` |
| `lockdown-core\src\recovery_log.rs` | `0x01AE81F5` | 19 | `lockdown-core\src\recovery_log.rs` |
| `lockdown-ipc\src\framing.rs` | `0x01AE59A4` | 9 | `lockdown-ipc\src\framing.rs` |
| `lockdown-ipc\src\client.rs` | `0x01AE4B9C` | 24 | `lockdown-ipc\src\client.rs` |
| `lockdown-ipc\src\server.rs` | `0x01AE5194` | 43 | `lockdown-ipc\src\server.rs` |
| `lockdown-ipc\src\codesign.rs` | `0x020CACD2` | 9 | `lockdown-ipc\src\codesign.rs` |
| `lockdown-windows\src\process\signing.rs` | `0x020D35F4` | 56 | `lockdown-windows\src\process\signing.rs` |
| `lockdown-shared\src\clipboard\clipboard_blocker.rs` | `0x01AE8CD0` | 12 | `lockdown-shared\src\clipboard\clipboard_blocker.rs` |

### 1.4 Operation handler modules — `lockdown-core\src\handlers\windows\` (30, client copy)

All 30 module-path literals, byte-exact in `digiexam.exe` (the *same 30 modules* are
also present in `service.exe` at the offsets in §2.3 — one operation engine, two hosts):

| Module | Offset | Module | Offset |
|---|---|---|---|
| `task_blocker.rs` | `0x01FC797C` | `rdp_blocker.rs` | `0x01FD34C4` |
| `serial_checker.rs` | `0x01FC79EC` | `service_starter.rs` | `0x01FD35DA` |
| `update_blocker.rs` | `0x01FC7A94` | `snapshot_collector.rs` | `0x01FD364C` |
| `audio_monitor.rs` | `0x01FC830A` | `power_button_blocker.rs` | `0x01FD373C` |
| `hotkey_blocker.rs` | `0x01FC8411` | `service_status_checker.rs` | `0x01FD37E9` |
| `process_reviver.rs` | `0x01FC8569` | `registry_sub_key_deleter.rs` | `0x01FD384C` |
| `service_stopper.rs` | `0x01FC8793` | `registry_string_value_setter.rs` | `0x01FD38B4` |
| `registry_value_deleter.rs` | `0x01FC880C` | `registry_integer_value_setter.rs` | `0x01FD3934` |
| `device_accessibility_check.rs` | `0x01FC88AD` | `registry_integer_value_verifier.rs` | `0x01FD39C8` |
| `service_existence_verifier.rs` | `0x01FC8972` | `registry_string_value_existence_verifier.rs` | `0x01FD3A60` |
| `registry_sub_key_existence_verifier.rs` | `0x01FC8A09` | `registry_integer_value_existence_verifier.rs` | `0x01FD3AD4` |
| `process_monitor.rs` | `0x01FDF2B4` | `sleep_blocker.rs` | `0x01FDF52D` |
| `vm_detector.rs` | `0x01FE33D6` | `dll_export_checker.rs` | `0x01FE349C` |
| `mac_address_checker.rs` | `0x01FE3519` | `file_existence_verifier.rs` | `0x01FE35A5` |
| `registry_string_value_verifier.rs` | `0x01FE3620` | `power_event_monitor.rs` | `0x01FEF524` |

*Handler names are descriptive only; what each operation does and with which
parameters is server-controlled at runtime (doc 13 §4) and the enforcement details
are deliberately not reproduced here (phase-1 report §4 holds the descriptive catalog).*

### 1.5 Win32 support layer — `lockdown-windows\src\` (32, client copy)

| Module | Offset | Module | Offset |
|---|---|---|---|
| `com.rs` | `0x01FC77CC` | `process\signing.rs` | `0x020D35F4` |
| `task_scheduler.rs` | `0x01FC7BD8` | `wmi.rs` | `0x020D8184` |
| `util.rs` | `0x01FC8A7C` | `service.rs` | `0x020D8214` |
| `user.rs` | `0x01FDBAFC` | `audio\mod.rs` | `0x020D84B2` |
| `process\snapshot.rs` | `0x01FF6C3C` | `power_management.rs` | `0x020D94BC` |
| `process\handle.rs` | `0x01FF7934` | `memory.rs` | `0x020F0432` |
| `vm_detection\generic\kernel_objects.rs` | `0x020CCE6A` | `windowing.rs` | `0x020F06CA` |
| `process\mod.rs` | `0x020CE63A` | `audio\device_manager.rs` | `0x020F25F4` |
| `vm_detection\x86_64\util.rs` | `0x020D1F12` | `update.rs` | `0x020F2B00` |
| `fs.rs` | `0x020D206A` | `registry.rs` | `0x020F2BC4` |
| `string.rs` | `0x020D2D4B` | `vm_detection\x86_64\timing.rs` | `0x020F2EDC` |
| `network.rs` | `0x020D2E14` | `vm_detection\x86_64\specific.rs` | `0x020F4862` |
| `audio\session_manager.rs` | `0x020F5D4C` | `keyboard.rs` | `0x020F5E8C` |
| `vm_detection\generic\ud.rs` | `0x020F6FEF` | `audio\monitor.rs` | `0x020F7C9C` |
| `audio\session.rs` | `0x020FDCA4` | `vm_detection\generic\nvram.rs` | `0x020FDCF4` |
| `vm_detection\generic\edid.rs` | `0x020FE04C` | `vm_detection\util.rs` | `0x02100F74` |

### 1.6 IPC channel names & message variants (client side)

| Raw literal | Offsets (all occurrences) |
|---|---|
| `lockdown_lite_ipc` | `0x01AE4B20`, `0x01AE5228`, `0x01B0E210`, `0x01B2F160`, `0x01E81280`, `0x01FF6190` |
| `digiexam_toolbox_service_ipc` | `0x01AE4F58`, `0x01B2E940` |
| `lockdown_windows_service_ipc` | `0x01AE4F90`, `0x01B158C8`, `0x01E75238` |
| `lockdown_notify_ipc` | `0x01AE51E0`, `0x01AF5FB8`, `0x01B29970`, `0x01B2F118`, `0x01E81238`, `0x01FC7738`, `0x01FD2A60`, `0x01FDEDA0`, `0x01FE1D70`, `0x01FEEC78`, `0x01FF6148`, `0x01FF9BA0` |
| `service_snapshot_collector_ipc` | `0x01FE7E30`, `0x01FF1470` |
| `RequestLockdownStart` | first `0x01B1B4C0` (+10; serde variant cluster `0x1B1C1BC–0x1FCFAED`) |
| `RequestLockdownStop` | `0x01B1C1F3`, `0x01FCFB10` |
| `RollbackOperations` | first `0x01B1B490` (+3) |
| `LockdownStopComplete` | `0x01B1C278`, `0x01FCFB35` |
| `NotifyAbort` | `0x01B1C28C`, `0x01FCFB49` |

**Raw snippet — pipe-name static table in `digiexam.exe` @`0x01AE4F30`**
(length/pointer pairs are Rust `&str` slice metadata; the names sit adjacent in
`.rdata`):

```
01AE4F30  6d 00 00 00 00 00 00 00 8e 00 00 00 2c 00 00 00  |m...........,...|
01AE4F40  80 59 ae 41 01 00 00 00 6d 00 00 00 00 00 00 00  |.Y.A....m.......|
01AE4F50  8d 00 00 00 21 00 00 00 64 69 67 69 65 78 61 6d  |....!...digiexam|
01AE4F60  5f 74 6f 6f 6c 62 6f 78 5f 73 65 72 76 69 63 65  |_toolbox_service|
01AE4F70  5f 69 70 63 00 00 00 00 90 57 ae 41 01 00 00 00  |_ipc.....W.A....|
01AE4F80  26 00 00 00 00 00 00 00 31 00 00 00 51 00 00 00  |&.......1...Q...|
01AE4F90  6c 6f 63 6b 64 6f 77 6e 5f 77 69 6e 64 6f 77 73  |lockdown_windows|
```

---

## 2. `service.exe` (Windows service) — lockdown components

### 2.1 App shell (`native\apps\lockdown-windows-service\src`)

| Module | Offset | Raw literal |
|---|---|---|
| `ipc.rs` | `0x00C624B0` | `native\apps\lockdown-windows-service\src\ipc.rs` |
| `lifecycle\mod.rs` | `0x00C63262` | `native\apps\lockdown-windows-service\src\lifecycle\mod.rs` |
| `lifecycle\snapshot_collector.rs` | `0x00C66578` | `native\apps\lockdown-windows-service\src\lifecycle\snapshot_collector.rs` |
| `service.rs` | `0x00C6CD5E` | `native\apps\lockdown-windows-service\src\service.rs` |
| `logging.rs` | `0x00C6D931` | `native\apps\lockdown-windows-service\src\logging.rs` |

Bare prefix `native\apps\lockdown-windows-service\src`: 42 occurrences (first `0x00C624B0`).

### 2.2 Shared crates (service copies)

| Crate module | First offset | n |
|---|---|---|
| `lockdown-core\src\handler\mod.rs` | `0x00C616FC` | 64 |
| `lockdown-core\src\handler_factory.rs` | `0x00C89687` | 1 |
| `lockdown-core\src\recovery_log.rs` | `0x00C61F0D` | 19 |
| `lockdown-ipc\src\codesign.rs` | `0x00D21ED2` | 9 |
| `lockdown-windows\src\process\signing.rs` | `0x00D89A34` | (same 32-module set as §1.5) |

### 2.3 Operation handler modules — 30, service copy

Same 30 module names as §1.4; service.exe offsets:

| Module | Offset | Module | Offset |
|---|---|---|---|
| `task_blocker.rs` | `0x00C883B4` | `rdp_blocker.rs` | `0x00C9101C` |
| `serial_checker.rs` | `0x00C88424` | `service_starter.rs` | `0x00C91132` |
| `update_blocker.rs` | `0x00C884CC` | `snapshot_collector.rs` | `0x00C911A4` |
| `audio_monitor.rs` | `0x00C88D42` | `power_button_blocker.rs` | `0x00C91294` |
| `hotkey_blocker.rs` | `0x00C88E49` | `service_status_checker.rs` | `0x00C91341` |
| `process_reviver.rs` | `0x00C88FA1` | `registry_sub_key_deleter.rs` | `0x00C913A4` |
| `service_stopper.rs` | `0x00C891CB` | `registry_string_value_setter.rs` | `0x00C9140C` |
| `registry_value_deleter.rs` | `0x00C89244` | `registry_integer_value_setter.rs` | `0x00C9148C` |
| `device_accessibility_check.rs` | `0x00C892E5` | `registry_integer_value_verifier.rs` | `0x00C91520` |
| `service_existence_verifier.rs` | `0x00C893AA` | `registry_string_value_existence_verifier.rs` | `0x00C915B8` |
| `registry_sub_key_existence_verifier.rs` | `0x00C89441` | `registry_integer_value_existence_verifier.rs` | `0x00C9162C` |
| `process_monitor.rs` | `0x00C9F8EC` | `sleep_blocker.rs` | `0x00C9FB65` |
| `power_event_monitor.rs` | `0x00CA9F1C` | `vm_detector.rs` | `0x00CB0DE6` |
| `dll_export_checker.rs` | `0x00CB0EAC` | `mac_address_checker.rs` | `0x00CB0F29` |
| `file_existence_verifier.rs` | `0x00CB0FB5` | `registry_string_value_verifier.rs` | `0x00CB1030` |

### 2.4 IPC names, peer validation, watchdog (service side)

| Raw literal | Offsets |
|---|---|
| `lockdown_lite_ipc` | `0x00C65350` |
| `lockdown_windows_service_ipc` | `0x00C61470`, `0x00C655F0`, `0x00C7D1E0` |
| `lockdown_notify_ipc` | first `0x00C61428` (+9) |
| `service_snapshot_collector_ipc` | `0x00C7D1FC`, `0x00CA2970`, `0x00CABE00` |
| `digiexam_toolbox_service_ipc` | **absent** (0 hits — the service never hosts/joins the toolbox channel) |
| `unauthorized IPC client attempted to connect` | `0x00C660A8` |
| `client passed validation` | `0x00C66180` |
| `double the heartbeat interval` | `0x00C61B2D`, `0x00C65B46` |
| `DigiexamWindowsService` | `0x00C6CD48`, `0x00C7DDD8` |
| `service_recovery_log` | `0x00C62558`, `0x00C75250` |
| `RequestLockdownStart` | first `0x00C6F28E` (+7) |
| `RequestLockdownStop` | `0x00C7FBDB`, `0x00C96EA8` |
| `RollbackOperations` | first `0x00C7EFD0` (+2) |

**Raw snippet — peer-validation strings in `service.exe` @`0x00C660A8`**
(the validator's log strings sit back-to-back in `.rdata`, followed by the
disconnect/cleanup pair):

```
00C660A0  45 01 00 00 11 00 00 00 75 6e 61 75 74 68 6f 72  |E.......unauthor|
00C660B0  69 7a 65 64 20 49 50 43 20 63 6c 69 65 6e 74 20  |ized IPC client |
00C660C0  61 74 74 65 6d 70 74 65 64 20 74 6f 20 63 6f 6e  |attempted to con|
00C660D0  6e 65 63 74 66 61 69 6c 65 64 20 74 6f 20 64 69  |nectfailed to di|
00C660E0  73 63 6f 6e 6e 65 63 74 20 70 69 70 65 20 69 6e  |sconnect pipe in|
00C660F0  73 74 61 6e 63 65 00 00 00 00 00 00 00 00 00 00  |stance..........|
00C66100  08 00 00 00 00 00 00 00 08 00 00 00 00 00 00 00  |................|
00C66110  80 4e 1b 40 01 00 00 00 66 61 69 6c 65 64 20 74  |.N.@....failed t|
00C66120  6f 20 73 69 67 6e 61 6c 20 74 68 61 74 20 6c 6f  |o signal that lo|
```

---

## 3. `toolbox-ipc-proxy.exe`

| Raw literal | Offsets | Note |
|---|---|---|
| `native\apps\toolbox-ipc-proxy\src\main.rs` | `0x00B72F4`, `0x00B7854`, `0x00B7DB4`, `0x00B9DCB` | thin stdio↔pipe forwarder (doc 09 §2.3) |
| `digiexam_toolbox_service_ipc` | `0x00B72D8`, `0x00B7838`, `0x00B7D98` | each adjacent to a `main.rs` panic path |

---

## 4. What is *not* recoverable / not published

1. **No source code exists in the binaries.** Rust release builds strip AST-level
   information; the module paths above come from panic-location metadata
   (`core::panic::Location`), which references *compiled* code but contains none.
2. **Machine code is not reproduced.** Dumping/disassembling enforcement functions
   would document implementation rather than location, which this project's scope
   excludes.
3. **Enforcement parameters excluded.** Registry values, hotkey lists, process
   whitelist mechanics and similar operational content are not reproduced anywhere in
   the published tree (phase-1 report §4 references them descriptively only).
4. **Function start addresses are not given.** Without shipping PDBs, file offsets of
   string literals cannot be mapped to function boundaries without disassembly —
   which would exceed the descriptive scope (and any RVA mapping would be
   version-fragile).

## 5. Reproduction

Reproduce every offset with the read-only scan script in
[`../extraction/TOOLING-SNIPPETS.md`](../extraction/TOOLING-SNIPPETS.md) §8 against
the hashed binaries in [`../recovered-assets/MANIFEST.md`](../recovered-assets/MANIFEST.md):

- `digiexam.exe` — SHA-256 `047BA68F640F…54E85491` (44,482,696 B)
- `service.exe` — SHA-256 `3EE5A430D6BF…96554D907E` (18,614,440 B)
- `toolbox-ipc-proxy.exe` — SHA-256 `05674FC5BBEF…2D0022B` (1,211,904 B)

All offsets in this document were produced by that exact method and spot-verified by
raw hexdump (§1.6, §2.4).
