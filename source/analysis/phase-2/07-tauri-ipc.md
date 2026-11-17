# 07 — Tauri IPC & capability model

**Phase:** 2 (2026-11-16). Official framework behavior verified against **Context7** (`/tauri-apps/tauri-docs`, Tauri v2 docs; `/tauri-apps/plugins-workspace` updater plugin) and distinguished from DigiExam-specific implementation. Labels: VERIFIED / STRONGLY INFERRED / UNVERIFIED.

## 1. Two distinct IPC layers — do not conflate

| Layer | Transport | Peers | Evidence |
|---|---|---|---|
| **Tauri IPC** (in-process, webview ↔ Rust) | Tauri v2 custom-protocol IPC (`ipc.localhost` HTTP-like scheme, JSON-serializable payloads) | WebView2 frontend ↔ `app_lib` Rust core | Context7 official docs; `__TAURI_INTERNALS__.invoke` in bundle (VERIFIED) |
| **Named-pipe IPC** (cross-process) | `\\.\pipe\` + protobuf-framed messages, peer code-signature validation | `digiexam.exe` ↔ `service.exe` ↔ `toolbox-ipc-proxy.exe` | Binary strings (see [`09-windows-integration.md`](09-windows-integration.md)) |

The research plan's "Electron IPC" question maps to **layer 1**; the client↔service
interfaces are **layer 2** — there is no Electron/Node process boundary anywhere
(NOT APPLICABLE).

## 2. Official Tauri v2 IPC behavior (Context7-verified)

- **Commands:** frontend `invoke()` ≈ fetch-like FFI abstraction; JSON-RPC-like
  serialization; all arguments/returns JSON-serializable.
- **Events:** fire-and-forget, one-way, bidirectional (frontend or Rust can emit);
  global or webview-targeted (`emit_to`); event *access* is itself permission-controlled;
  events do **not** carry the fine-grained capabilities/typing that commands have.
- **Channels:** streaming Rust→JS data (used by DigiExam for `DownloadEvent`).
- v2 IPC runs over custom protocols rather than `postMessage` for performance.

**DigiExam-specific usage — VERIFIED (bundle + binary command tables):**
- App commands: `check_update`, `download_update`, `install_update`, `autoupdate`,
  `exit`, `restart`, `set_server_uri`, `start/stop_exam_connectivity`,
  `start/stop_countdown_timer`, `save_textual_answer_stream`, `rollback_from_recovery_log`,
  logger commands (`send_debug/trace/info/warn/error`), token commands.
- Plugin commands: `plugin:lockdown-lite|run_all/stop_all/rollback/query_windows_service_status/...`,
  `plugin:webview-manager|open_webview/...` (full registry in
  [`04-frontend-ipc-surface.md`](../phase-1/04-frontend-ipc-surface.md)).
- Rust→frontend events: `lockdown_started`, `lockdown_stopped`, `critical_lockdown_error`,
  `action_required_error`, `abort`, `network_ping`, permission checks, `battery_level`,
  `available_disks`.

## 3. Capability/ACL model (Tauri v2) — official semantics vs observed table

**Official (Context7):**
- Capabilities bind *windows/webviews* → *permissions*; permissions enable/deny
  commands and can attach scopes (`allow`/`deny` sets).
- Custom commands registered via `invoke_handler` are **allowed by default** unless
  restricted via `AppManifest::commands` in `build.rs`.
- Permission identifiers follow `allow-<kebab>` / `deny-<kebab>`; events are covered by
  core event permissions.

**Observed in `digiexam.exe` — VERIFIED:** an embedded ACL table (offset region
≈33,070,000–33,076,000) with exactly the official identifier grammar (`allow-…` /
`deny-…`, plugin command names), covering core window/webview commands, lockdown-lite,
io, battery, keyboard-layout, updater (details in
[`04-frontend-ipc-surface.md`](../phase-1/04-frontend-ipc-surface.md) §4).

**Interpretation (STRONGLY INFERRED):** DigiExam uses the standard v2 ACL mechanism;
the table is the compiled form of capabilities/permission files. Which commands the
*shipped webview configuration* can actually exercise at runtime is **UNVERIFIED**
(static ACL presence ≠ runtime grant state).

## 4. Notable ACL observations (descriptive)

- `allow-internal-toggle-devtools` exists in the allow-list (phase-1 §8.2) — an
  ACL-level devtools surface; runtime exposure **UNVERIFIED**.
- `deny-` entries exist for some commands in some scopes (e.g. `set_content_protected`
  denied in certain scopes) — a least-privilege tightening pattern consistent with
  official guidance.

## 5. Tauri updater plugin — official contract vs DigiExam wiring

**Official (Context7, plugins-workspace):** `Update::download_and_install` verifies the
package signature (minisign) then, **on Windows, launches the installer and exits the
app** (`std::process::exit(0)`); macOS/Linux require explicit relaunch.

**DigiExam (VERIFIED):** matches the official Windows contract — frontend
`autoupdate` short-circuit, `msiexec … /passive /promptrestart`/NSIS `/UPDATE` installer
strings, CrabNebula update endpoint and embedded minisign key (see
[`10-backend-protocols.md`](10-backend-protocols.md) §4). No custom updater patching
observed.

## 6. Open questions (UNVERIFIED)

- Runtime-effective grants vs the static ACL table.
- Whether any second webview (child exam webview) has a *different* capability set
  than the main window.
