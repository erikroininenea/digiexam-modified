# Rapport: Hur Digiexam Lockdown-läge fungerar (Windows)

**Mål:** `D:\Digiexam\digiexam.exe` (v26.4.7) + `service.exe` + `toolbox-ipc-proxy.exe`
**Metod:** Sträng-/konstantanalys av native-binärer samt försök med dnSpy (se nedan). Endast läsning – inga patchar eller modifieringar gjordes.

---

## 1. Sammanfattning (TL;DR)

Digiexam på Windows är **inte .NET** – det är en **Tauri-app (Rust + WebView2)**. därför kan dnSpy
inte ladda filerna (verifierat: PE x64 native, inget CLR-metadata i någon av de tre binärerna).
Själva låsningen ligger i `service.exe`, som körs som Windows-tjänsten **"DigiexamWindowsService"**
(mjukvara skriven i Rust, moduler `lockdown-windows-service`, `lockdown-core`, `lockdown-windows`).

Låsning sker genom ett **operationsmodell** (Server-driven):

1. Digiexam-klienten (Tauri) hämtar en **lås-konfiguration över gRPC**
   (`lockdown.service.v1.LockdownService/GetLockdownConfig`) från Digiexams backend (`*.digiexam.com`).
2. Klienten skickar konfigurationen + sessionsinfo (`organization_id, user_id, exam_id, is_proctored`,
   OAuth-tokens, `client_session_id`) till tjänsten via **named pipes IPC**
   (`lockdown_windows_service_ipc`).
3. Tjänsten exekverar en lista av **operationer** (blockerare, verifierare, processmonitor, registry-ändringar osv.),
   registrerar var och en i en **recovery-log** på disk så att allt kan **återställas (rollback)**.
4. Ömsesidig **watchdog/heartbeat**: om klienten dör utan att ställa av lockdown rullar tjänsten tillbaka allt
   automatiskt; om tjänsten dör har klienten notifier-servern för abort, och recovery-loggen säkerställer
   återställning vid omstart.

---

## 2. Komponenter & roller

| Binär | Roll (från strängar) |
|---|---|
| `digiexam.exe` (44 MB) | Tauri 2.x-app: UI i WebView2, gRPC-klient mot backend, updater (tauri-plugin-updater), `"lockdown-lite"`-plugin, startar `toolbox-ipc-proxy.exe`. |
| `service.exe` (18 MB) | Windows-tjänst `DigiexamWindowsService` med executor: startar/stannar/rollbackar operationer, watchdog, recovery-log, gRPC `GetLockdownConfig`-fetch ("legacy" flöde). |
| `toolbox-ipc-proxy.exe` | IPC-proxy ("digiexam_toolbox_service_ipc", "toolbox proxy stopped being available"), startas via stdin-pipe från klienten; updater/tjänste-relaterad broker. |

Namngivna IPC-kanaler (named pipes, `\\.\pipe\...`):
`lockdown_windows_service_ipc`, `lockdown_notify_ipc`, `lockdown_lite_ipc`,
`service_snapshot_collector_ipc`, `digiexam_toolbox_service_ipc`.

Protokoll (tonic/gRPC + protobuf, från protobuf-service-rader):
- `lockdown.service.v1.LockdownService`: `GetLockdownConfig`, `LockdownStart`,
  `LockdownStartWithDefinitions`, `LockdownStop`, `RollbackOperations`.
- `assignment.service.v1.AssignmentService/SetConnectivityStatus` (klienten rapporterar exam-konnektivitet med examId/studentId).
- `lockdown.opdefinition.v1`, `lockdown.lifecycle.v1`, `lockdown.parameter.v1` (definitions-/parametermodell).

---

## 3. Livscykeln för lockdown

**Start**
- `"legacy lockdown start request received; fetching executor configuration"` – tjänsten kan själva fetcha config via `GetLockdownConfig`.
- `"initializing lockdown from client-supplied configuration"` – normalfallet: klienten levererar serverns
  operation-definitioner (`OperationDefinition`, `ApplicableOperation`, `Parameter`) med filter på OS-id,
  arch, min/max-version, `require_administrator`, `valid_from/valid_to`.
- `"Lockdown already running"`, `"acknowledging replayed lockdown start request"` – idempotens vid replay.
- SessionInfo präglar operationerna: `organization_id, user_id, exam_id, is_proctored`.

**Körning**
- Varje operation startas → `"operation started: ?"`, registreras i recovery-log.
- `"OperationStarted"` / `"OperationRolledBack"` / `"Error"` / `"ActionRequired"` returneras som `ExecutionResult`.
- Status kan vara `ManualProcessKillingRequired` / `OperationRequiresUserAction` om något inte kan dödas automatiskt.

**Stopp / crash-säkerhet**
- `"lockdown stop requested"` → varje operation stoppas, rollback skapas: `"Created rollback operation for ?"`.
- Watchdog i tjänsten: `"client failed to respond within 4 times the heartbeat interval, so we presume it has died"`
  → abort/rollback om klienten hänger sig eller stängs.
- Klientens sida: `"lost connection with watchdog"`, `"notify server failed to abort lockdown"` – klienten kan
  begära abort via `lockdown_notify_ipc`.
- `recovery_log.rs`: persistens av vilka operationer som är igång, så att en crashad tjänst
  `"failed to find handler for rollback operation in recovery log"` inte tappar återställningen. Loggen
  rensas vid lyckat stopp (`"failed to clear lockdown log"`).

---

## 4. Operationskatalogen (de konkreta låsåtgärderna)

Utifrån `default_operations.rs` + `default_operation_definitions.rs` + handler-modulerna under
`native\libs\lockdown-core\src\handlers\windows\`:

**Blockerare (aktiv låsning)**
- `WindowsHotkeyBlocker` – *"Block hotkeys on Windows"*, parameter `Hotkeys to block`; kompletterad av
  low-level **keyboard hook** (`lockdown-windows/src/keyboard.rs`, `SetWindowsHookEx`-mönster:
  "failed to install keyboard hook", "passing to next hook"). Hotkey-visningsnamn i binären: `Ctrl+Alt…`,
  `Ctrl+Shift…`, `Win+Shift…`, `Alt+PrintScr`, `Win+Plus`, `Alt+Left`, `Ctrl+Win`, `PrintScr`.
- `SleepBlocker` – *"Block sleep on Windows"* (sömn/vänteläge blockeras).
- `PowerButtonBlocker` – *"Disable power buttons"* (av/på-knappar i OS-nivå).
- `PowerEventMonitor` – *"Stop lockdown if system goes to sleep or lid is closed"* (säkerhetsventil om blockeraren misslyckas).
- `RdpBlocker` – RDP-sessioner nekas: `"session protocol type is RDP"`, `"a session is shadowing another session"`
  (blockerar fjärrstyrning/skärmdelning av konsolsessionen).
- `TaskBlocker` – *"Block tasks on Windows"* (inaktiverar schemalagda uppgifter: "Failed to disable tasks for operation ?").
- `UpdateBlocker` – *"Block updates on Windows"* (Windows Update stoppas under provet).
- `ServiceStopper` / `ServiceStarter` / `ServiceStatusChecker` – *"Stop a service"*, *"Start a service"*.
- `RegistryIntegerValueSetter` / `RegistrySetStringValue` / `RegistryValueDeleter` / `RegistrySubKeyDeleter` +
  verifierare (`RegistryIntegerValueExistenceVerifier`, `RegistryStringValueExistenceVerifier`,
  `RegistrySubKeyExistenceVerifier`, `FileExistenceVerifier`, `DllExportChecker`, `DeviceAccessibilityChecker`,
  `ServiceExistenceVerifier`) – *"Set/Verify/Delete a registry value/sub key"*.

**Register-/policyblockeringar som default-konfig (bokstavligt ur binären):**
| Nyckel | Värde | Effekt |
|---|---|---|
| `HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer` | `DisableTaskMgr` | Task Manager avstängd |
| samma | `NoLogoff`, `NoDriveTypeAutoRun` | Logga ut/Autorun blockerade |
| `HKCU\...\Policies\System` | `DisableChangePassword`, `DisableLockWorkstation` | Lösenordsbyte, Win+L |
| `HKLM\...` | `HideFastUserSwitching` | snabbbyten dolda |
| `Software\Microsoft\input\Settings` | `EnableClipboardHistory` = 0 | Urklipps-historik av |
| `HKLM\...\Image File Execution Options\psexec.exe` | `Debugger` = `svchost.exe` | **psexec ges falsk debugger-process** – nekades köras under provet |

**Processbevakning (anti-cheat, systemnivå)**
- `ProcessMonitor` – *"Monitor processes on Windows"*. Parametrar: `allowed_signer_signatures`
  ("Executable signer certificates allowed to run") och `allowed_unsigned_hashes`
  ("SHA-256 hashes of whitelisted files allowed to be unsigned"). Dvs **vitelist på Authenticode-signerare**:
  alla processer som inte matchar dödas: `kill_tree`, `"process killed"`, `taskkill.exe /PID /F`,
  `"illegal parent found"` → `"failed to kill illegal parent"` (förälderprocessen dödas också).
  `"process signature fingerprint didn't match any of the allowed ones"`.
- `ProcessReviver` – *"Restart killed processes on Windows"*, parameter `Processes to start`;
  systemkomponenten `explorer.exe` dödas (taskkill) och **återstartas automatiskt** för att tillfälligt
  brända systemprocesser ska återhämta sig i rätt låst state.
- `SnapshotCollector` – *"Capture system snapshot"* (v1.0.0): full hardware/software/network/virtualization-
  inventering via WMI (`Win32_BIOS`, `Win32_OperatingSystem`, `Win32_NetworkAdapterConfiguration` med
  `MACAddress`, `IPAddress`, `DNSServerSearchOrder`, …), fält som `hypervisor`, `manufacturer`, `model`,
  `storage`, `memory`, `graphics`, `os_architecture` m.m. – prov-miljön dokumenteras.

**Detektorer (rapporterar, dödar inte nödvändigtvis)**
- `VmDetector` – "VM substring detected in brand" för `qemu, kvm, vbox, virtualbox, monitor, bhyve,
  hypervisor, hvisor, parallels, vmware` (m.a.o. ljud-, CPU-, NVRAM-brandsträngar: `"NVRAM: QEMU/OVMF detected"`,
  `"Hyper-V VM detected"`, timer-tester `rdtscp`, `"Cannot run timer test without rdtscp!"`).
- `MacAddressChecker` – *"Check if MAC addresses are present"* – verifierar mot kännd MAC-lista (t.ex.
  skolans nät/adapter), parameter `MAC addresses to check`.
- `SerialChecker` – *"Check serials"* – verifierar seriell-hårdvara (parameter `Serials to check`).
- `WindowsAudioMonitor` – lyssnar på audio-device-brand mot VM-listan (själva detekteringen ligger i
  `lockdown-windows/src/audio/device_manager.rs`).
- `PowerEventMonitor` (ovan), `ServiceExistenceVerifier`, `DeviceAccessibilityChecker` (t.ex. USB-/device-path).

**Tauri/UI-nivå (i `digiexam.exe`, plugin `tauri-plugin-lockdown-lite` + `webview-manager`)**
- Fullskärms-enforcement: `"window unexpectedly exited fullscreen whilst in lockdown"`.
- Clipboard-monitor i Windows-komponent (`setup_component/windows/clipboard_monitor.rs`).
- `set_clipboard_text`, `open_automation`-commands, clipboard-text-block, "exam connectivity report"
  → backend `SetConnectivityStatus`.
- Webview hardening: `clear_all_browsing_data`, `set_content_protected` (blockar skärminspelning/dump av webview-innehåll).

**Självskydd av tjänsten**
- Process mitigations vid start (`SetProcessMitigationPolicy`): *"image load mitigation policy"*
  (blockerar icke-signerade DLL:er/remote-images) och *"extension point policy mitigation"*
  (AppInit-DLL:er/shell-extensions) – dvs anti-injektion-försvar för tjänstprocessen.

---

## 5. Flödet i en provsession (sammanfattningsbild)

```
Digiexam backend (*.digiexam.com, gRPC TLS, OAuth v2 token-endpoint)
   ▲ GetLockdownConfig (operation-definitioner per org/OS/ark/version)
   │
digiexam.exe (Tauri/WebView2)
   │  named pipe: lockdown_windows_service_ipc  (+ lockdown_notify_ipc)
   │  payload: OperationDefinition[] + SessionInfo{org, user, exam, is_proctored, tokens}
   ▼
DigiexamWindowsService (service.exe, SYSTEM)
   ├─ executor: startar Operation[]  ── recovery_log (disk) ──┐
   ├─ watchdog: heartbeat → klient död ⇒ abort + rollback ◄──┘
   ├─ handlers: registry, hotkeys, sleep, power, RDP, tasks, updates, services
   ├─ ProcessMonitor (signerings-whitelist) + ProcessReviver + SnapshotCollector (WMI)
   ▼
Systemet låst under provet; stopp = rollback i omvänd ordning + logg rensad.
```

---

## 6. Observationer / begränsningar i analysen

- **dnSpy MCP var inte användbart** – inga .NET-moduler i paketet (verifierat: MZ/PE x64 native, ingen
  BSJB/CLR-metadata). Rust-symbolvägar i binärerna (`native\apps\..., native\libs\...`) ersätter det:
  de ger exakta modulnamn, handler-ID:n och protobuf-schema.
- Operation-seten som faktiskt körs vid ett prov är **serverstyrd** (per organisation, OS-id, arch,
  version-intervall, `require_administrator`, `valid_from/to`) – default-katalogen ovan är vad binären
  bär inbyggt; "Lite"-läget (`lockdown_lite_ipc`) är en enklare klient-intern väg utan full serviceexecutor.
- Käll-IP/endpoint för `GetLockdownConfig` i "legacy"-flödet: `https://*.digiexam.com`
  (även `auth.`/`telemetry.`-subdomäner, OAuth `oauth/v2/token`).
- Rapporten beskriver **observationer i koden**; ingen modifiering, patchning eller försök att kringgå
  låset har gjorts. Autentiserade låsbegäranden (tokens, session-id) och service-signering indikerar att
  tjänsten förväntar sig en auktoriserad klient (klientens PID/token tas emot via named pipe).

---

## 7. Djupdyk: Frontend-analys (fas 2, "REA-spår")

### 7.1 Elektroteknisk utredning: är det Electron? — **Nej.**
- De enda 2 träffarna på "Electron" i `digiexam.exe` är falska positiva: "Electron**ic**" i en vanlig-ordlista-blob (offset 34276074) och "Secure **Electronic** Transactions" i X.509-OID-tabell (36344983).
- Definitiva Tauri-signaturer: `tauri-2.10.3\src\ipc\command.rs`, `tauri-plugin-updater-2.10.1`, `tauri-plugin-deep-link-2.4.7`, `tauri-plugin-store-2.4.2`, `__tauri`-symboler, `window.__TAURI_INTERNALS__` i inline-JS.
- **Slutsats (observation):** Digiexam v26.4.7 Windows = **Tauri 2.10.3 + WebView2**, inte Electron/Chromium.

### 7.2 Extraktion av frontend-assets ur binären
Tauri release-byggen brotli-komprimerar embedded assets. Asset-tabellen hittades via sökvägssträngar
(`/assets/index-CvNvudRa.css`, `/assets/KaTeX_*.woff2`, `g/index.html` …), och brotli-strömmar lokaliserades
genom riktad brute-force-scanning med Node `zlib.brotliDecompressSync` (ingen magisk signatur finns, men
dekomprimeringen misslyckas snabbt på skräp – ~30 000 offsets tog 16 ms).

| Asset | Offset i exe | Dekomprimerad storlek |
|---|---|---|
| `index.html` | 30807145 | 1 167 B (referera `/assets/index-COO2fUbz.js` + CSS) |
| `assets/index-COO2fUbz.js` | 28509689 | **10 547 069 B** (frontend-bundel, Vite/esbuild, React + Redux Toolkit) |
| `assets/index-CvNvudRa.css` | 31489343 | 144 919 B |

Extraherat träd: `%TEMP%\opencode\digiexam_frontend\` (index.html + assets/).

### 7.3 Frontendens kompletta IPC-yta (INVOKE/LISTEN-registret ur bundeln)
Frontend → Rust-kommandon (bokstavligt ur `INVOKE`-objektet):
- **LOCKDOWN:** `plugin:lockdown-lite|run_all` (START), `stop_all`, `rollback`, `query_windows_service_status`
- **SECURITY:** `open_accessibility`, `open_automation`, `open_full_disk`, `open_network_extensions`,
  `open_content_filter` (macOS-behörigheter), `rollback_from_recovery_log`
- **CLIPBOARD:** `set_clipboard_text`, `clear_clipboard`
- **WEBVIEW:** `open_webview`, `hide_webview(s)`, `close_webview(s)`, `navigate_home/back/forward`, `refresh`, `update_ratio`, `get_selected_text`
- **Övrigt:** `start/stop_exam_connectivity`, `save_textual_answer_stream`, `start/stop_countdown_timer`,
  `check/download/install_update`, `exit`, `restart`, `set_server_uri`, `start_ping`, logger-kommandon,
  `plugin:battery|*`, `plugin:io|*` (USB-endpoint-monitoring, `read_to_dxe`, audio files),
  `plugin:keyboard-layout|get/set_keyboard_layouts`

Händelser (Rust → frontend): `lockdown_started`, `lockdown_stopped`, `critical_lockdown_error`,
`action_required_error`, `abort`, behörighetskontroller (`accessibility_check`, `automation_check`,
`full_disk_check`, `sysext_check`, `filter_check`), `battery_level`, `available_disks`, `network_ping`.

### 7.4 Start-/stoppflödet i frontend (Redux-thunks)
```
startOnlineExam (thunk)
 ├─ startLockdown(orgId, studentId, examId,
 │    lockdown = !openBook, clipboard = allowExternalPaste, isProctored = remoteProctoring)
 │    └─ invoke("plugin:lockdown-lite|run_all", {...})
 │       + createEventWaiter(once "lockdown_started", fel: critical/action_required/abort)
 ├─ getAssignmentService().startOnlineExam(examId, studentId)   // gRPC mot backend
 └─ vid fel: stopLockdown(!openBook, allowExternalPaste)        // auto-rollback
```
- **`openBook`-prov → `lockdown=false`** (demonstrationsexamen är `openBook: true` → endast lite-läge).
- `allowExternalPaste` styr klippbordsfunktionalitet under provet.
- `remoteProctoring` flaggar proctoring-läge (50/213 träff på "proctor"/"Proctor" i UI-koden).

`dispatchLockdownMessage` (från Rust/WASM → JS): `"LockdownStarted"` / `"LockdownStopped"` eller
objekt med `CriticalLockdownError`, `OperationFailed`, `ActionRequired`, `Abort` → mappas till
`critical_lockdown_error` / `action_required_error` / `abort`-händelser.

### 7.5 Tre driftlägen (PLATFORM)
`PLATFORM = { NATIVE: "native", WEB: "web", WASM: "wasm" }` — samma React-app körs i:
1. **native** – Tauri-appen (denna binär); lockdown via `lockdown-lite`-plugin → `service.exe`.
2. **wasm** – webbläsarvariant med WASM-modul som får `initialize(dispatchLockdownMessage)` + Chrome-extension
   ("IWA") för USB-lagring och keep-awake; wasm-modulen tar dispatch-roller.
3. **web** – ren webbläsarläge (öppna prov utan lockdown).

### 7.6 REA-verktygsanvändning (CLI-fallback)
- REA MCP-server var ej registrerad; CLI-fallback `npx -y rea-agents@6.9.0` användes.
- **Körning 1** (default heap): JS-analysatorns V8-heap (1024 MiB) tog slut på den 10,5 MB bundeln → `parse_status: "skipped"`. Inventory + HTML-parse genomfördes; limitation explicit i Evidence `ev_7814870635e0…`.
- **Körning 2** (`--max-heap-mb 4096 --detail summary`): heap-gränsen löst, men per-käll-timeouten (300 s) stoppade djupanalysen av samma fil (`skipped_javascript_files: 1`, `parsed_javascript_files: 0`, timeout-limit `javascript_analysis_worker_timeout_ms = 300000`). Del-evidence returerades: `ev_b5e2fe93a6b7…` (parent), summary-projektion `ev_12d550e41d…`, bundle sparad i `%TEMP%\opencode\rea_bundle`.
- **REA:s Electron-check stöder slutledningen:** alla Electron-fält (browser_windows, preload_entrypoints, context_bridge_apis, ipc-operations, native_addon_bindings …) = **0** — arten är inte Electron. `relevant_files: 2`, `text_bytes_read: 10 548 236`.
- Slutsats om REA-kapacitet: en monolitisk 10,5 MB minifierad bundel ligger utanför REA:s per-källbudgetar i båda led (heap och tid); en full semantisk graf kräver längre `--analysis-timeout-ms` eller uppdelning av bundeln. Observationerna i 7.3–7.5 är tagna direkt ur den läsbara esbuild-utskriften.

---

## 8. Rust-kärnans kommandoregister & IPC-förtroendemodell (fas 3)

### 8.1 Plattformskorrigering
Projektets ursprungsa antagande om "Electron Main/Renderer Process" är **felaktigt och belagt så**:
DigiExam v26.4.7 är **Tauri 2.10.3 + WebView2**. Motsvarigheterna är:
- "Main Process" → **Rust-kärnan i `digiexam.exe`** (`native\apps\digiexam\src\lib.rs` + `command\*.rs`)
- "Renderer Process" → **WebView2-webviewen** med frontend-bundeln (`assets/index-COO2fUbz.js`)
- "Native Node-moduler" → finns ej; rollen fylls av Tauri-plugins i Rust (lockdown-lite, webview-manager, io, battery, keyboard-layout, store, updater, deep-link).

### 8.2 Verifierat: Tauri v2 capability/ACL-system används
Binären innehåller Tauris permission-ACL-tabell (`allow-<kebab>`/`deny-<kebab>` + kommandonamn i snake_case,
offset-region ~33 070 000–33 076 000). Det visar att frontends IPC-yta är **begränsad per plugin**:

| Plugin | Tillåtna kommandon (ur ACL) | Några explicit deny |
|---|---|---|
| `core` (window) | get_all_webviews, webview_position/size, show/hide/close, clear_all_browsing_data, **internal_toggle_devtools**, is_fullscreen, available_monitors … | set_content_protected (deny!), set_fullscreen, set_decorations, start_dragging |
| `core` (webview) | create_webview(_window), set_webview_focus/position/size/zoom … | create-webview (deny i vissa scopes), internal-toggle-devtools |
| `lockdown-lite` | **run_all, stop_all, rollback, query_windows_service_status**, set/clear_clipboard, open_accessibility/automation/full_disk/network_extensions/content_filter | clear-clipboard/query-windows-service-status/set-clipboard-text (deny i vissa scopes) |
| `io` | start/stop_endpoint_monitoring, read_to_dxe, list_audio_files, read_audio_file | (motsvarande deny-poster finns) |
| `battery` | has_battery, start/stop_battery_monitoring | — |
| `keyboard-layout` | get/set_keyboard_layouts | — |
| `updater` | **download_and_install**, exe-extension | bundle-type (deny) |

**Observation (viktig, verifierad):** `allow-internal-toggle-devtools` finns i allow-listan — devtools-toggle
exponeras mot frontend. Att devtools sedan är stängt i release styrs av Tauri-webviewkonfiguration, ej ACL.

### 8.3 Appens egna kommandon (verifierade strängar vid ~28 212 000)
`rollback_from_recovery_log`, `autoupdate`, `install_update`, `download_update`,
`start_exam_connectivity`, `stop_exam_connectivity`, `start_countdown_timer`, `stop_countdown_timer`,
`insert_metadata`, `insert_many_metadata`, `remove_metadata`, `remove_many_metadata`, `clear_metadata`
(logger-stream till Grafana-backend), `set_server_uri` (region.rs), samt updater-flödet
(`DownloadEvent::Started/Progress/Finished`).

### 8.4 Ny verifierad säkerhetsmekanik: **client-side IPC-validering**
Ur `digiexam.exe` (lib.rs / lockdown-core-regionen):
- `"unauthorized IPC client attempted to connect"` — **även klientens IPC-server validerar anslutande processer**
- `"client passed validation"`
- `"notified windows service of session ID"` / `"failed to send client update to windows service"` — klienten etablerar session mot tjänsten
- `"failed to reach windows service during startup"`, `"...region"` (region.rs) — region/server-URI propageras till tjänsten
- `"Connected to IPC server at '...'"` + `recovery_log.rs` **i klienten** — klienten bäddar också in lockdown-core (har rollback-logik lokalt, inte bara tjänsten)
- `native\libs\lockdown-shared\src\clipboard\clipboard_blocker.rs` — delad klippbordsblockerare i klienten
- Timer: `"sending timer update"`, `"timer elapsed!"`, `"timer cancelled"` (Rust-sidans provtimer)
- TTS-bridge i webview-manager (`"tts: using the bridge selection"`)

**Förtroendemodellen är därmed dubbelriktad:** både tjänsten (`service.exe`) och klientens IPC-server
validerar motparten vid pipe-anslutning ("unauthorized IPC client…" / "client passed validation"),
och klienten meddelar tjänsten session-ID + region vid uppstart.

### 8.5 Hypoteser (konceptuell säkerhetsdiskussion — ej verifierade)
Nedan är designobservationer på konceptnivå; varje punkt kräver verifiering i isolerad, auktoriserad miljö
(och i praktiken koordinering med Digiexam) — **inga kringgående-metoder dokumenteras här**:
- **H1 (konfig-proveniens):** låskonfig laddas av klienten (`GetLockdownConfig`) och skickas till tjänsten
  ("client-supplied configuration"). HYPOTES: mediatrösten ligger i att konfigens innehåll kontrolleras av backend;
  verifierbar fråga i labb: om tjänsten validerar att konfigens signatur/token matchar sessionen, eller
  accepterar vilken autentiserad klient som helst.
- **H2 (watchdog-marginer):** abort vid 4× heartbeat-intervall. HYPOTES: beteende vid tillfälliga hänger
  (t.ex. GC/energisparläge) — verifieras genom lasttest i labb.
- **H3 (rollback-slutenhet):** recovery-log garanterar återställning efter crash. HYPOTES: oordnad
  avstängning (strömavbrott) mellan operation-start och logg-skriv → konceptuellt kan en ändring släppa
  igenom utan rollback. Labb: crash-injektion vid exakta tidpunkter.
- **H4 (devtools-exponering):** `internal_toggle_devtools` i allow-lista. Konceptuellt: om webviewens
  devtools är aktiverade i release finns en felsökningsyta. Verifiering: inspektera webviewkonfig i labb.

Verifiering av samtliga hypoteser kräver en isolerad testmiljö med Digiexams medgivande (labsession/
testorganisation), eftersom verifieringsprotokollen i sig skulle röra låsens tvingande delar.

*Bilagor: %TEMP%\opencode\strings_service.txt, strings_digiexam.txt, strings_toolbox.txt; frontend-träd %TEMP%\opencode\digiexam_frontend\; REA-bundle %TEMP%\opencode\rea_bundle.*
