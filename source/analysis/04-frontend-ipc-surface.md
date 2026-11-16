# 04 — Frontend IPC surface (recovered bundle analysis)

**Source:** `/assets/index-COO2fUbz.js` (Vite/esbuild bundle, 10,547,069 B), recovered
from `digiexam.exe` — see [`../recovered-assets/MANIFEST.md`](../recovered-assets/MANIFEST.md).
All items below are **observations from static text of the bundle**; no application was
executed.

## 1. Tauri command surface invoked by the frontend

The bundle's `INVOKE` registry enumerates every Tauri command the UI can call
(plugin-prefixed):

| Domain | Commands (observed) |
|---|---|
| **Lockdown** | `plugin:lockdown-lite\|run_all` (START), `stop_all`, `rollback`, `query_windows_service_status` |
| **Security/permissions** | `open_accessibility`, `open_automation`, `open_full_disk`, `open_network_extensions`, `open_content_filter`, `rollback_from_recovery_log` |
| **Clipboard** | `set_clipboard_text`, `clear_clipboard` |
| **Webview** | `open_webview`, `hide_webview(s)`, `close_webview(s)`, `navigate_home/back/forward`, `refresh`, `update_ratio`, `get_selected_text` |
| **Exam session** | `start/stop_exam_connectivity`, `save_textual_answer_stream`, `start/stop_countdown_timer` |
| **Updater** | `check/download/install_update` |
| **App control** | `exit`, `restart`, `set_server_uri`, `start_ping` |
| **Plugins** | `plugin:battery\|*`, `plugin:io\|*` (USB endpoint monitoring, `read_to_dxe`, audio files), `plugin:keyboard-layout\|get/set_keyboard_layouts` |

## 2. Events emitted Rust → frontend

`lockdown_started`, `lockdown_stopped`, `critical_lockdown_error`,
`action_required_error`, `abort`, permission checks (`accessibility_check`,
`automation_check`, `full_disk_check`, `sysext_check`, `filter_check`),
`battery_level`, `available_disks`, `network_ping`.

## 3. Exam-start control flow (Redux thunk, as readable in the bundle)

```
startOnlineExam (thunk)
 ├─ startLockdown(orgId, studentId, examId,
 │    lockdown = !openBook, clipboard = allowExternalPaste,
 │    isProctored = remoteProctoring)
 │    └─ invoke("plugin:lockdown-lite|run_all", {...})
 │       + createEventWaiter(once "lockdown_started";
 │         errors: critical_lockdown_error / action_required_error / abort)
 ├─ getAssignmentService().startOnlineExam(examId, studentId)   // gRPC → backend
 └─ on failure: stopLockdown(...)                               // auto-rollback
```

**Observations:** an `openBook` exam maps to `lockdown=false` (demonstration exams run
in lite mode); `allowExternalPaste` gates clipboard functionality during the exam;
`remoteProctoring` flags proctored sessions (50 hits of "proctor"/"Proctor" in UI code).

## 4. Capability/ACL system (Tauri v2)

The binary embeds Tauri's permission ACL table (`allow-<kebab>` / `deny-<kebab>` +
snake_case command names, offset region ≈ 33,070,000–33,076,000). Summary per plugin
(phase-1 §8.2):

| Plugin | Allowed (excerpt) | Explicitly denied in some scopes |
|---|---|---|
| `core` (window/webview) | `get_all_webviews`, position/size, show/hide/close, `clear_all_browsing_data`, `internal_toggle_devtools`, `is_fullscreen`, `available_monitors` | `set_content_protected`, `set_fullscreen`, `set_decorations`, `start_dragging`, some `create-webview` scopes |
| `lockdown-lite` | `run_all`, `stop_all`, `rollback`, `query_windows_service_status`, clipboard set/clear, `open_*` permission dialogs | clear-clipboard / status / set-clipboard-text in some scopes |
| `io` | endpoint monitoring start/stop, `read_to_dxe`, audio file listing/reading | corresponding denies |
| `battery` | `has_battery`, battery monitoring start/stop | — |
| `keyboard-layout` | `get/set_keyboard_layouts` | — |
| `updater` | `download_and_install`, `exe` extension | `bundle-type` |

**Notable observation:** `allow-internal-toggle-devtools` appears in the allow-list —
a devtools toggle surface is exposed to the frontend ACL-wise. Whether the release
webview actually enables devtools is a webview-config question and was **not verified
at runtime** (recorded as an open item in phase-1 H4).

## 5. Backend endpoints referenced by the frontend/Rust layers

- `https://*.digiexam.com` (incl. `auth.` and `telemetry.` subdomains)
- OAuth: `oauth/v2/token`
- gRPC: see [`03-component-architecture.md`](03-component-architecture.md) §2

## 6. Unknowns

- Runtime behavior of the ACL in the shipped webview configuration (allow vs. effective)
- Exact split of lockdown operation execution between `lockdown-lite` (client-internal)
  and the full service executor per exam type — the *distribution* is server-defined
  and only observable in a live, authorized session.
