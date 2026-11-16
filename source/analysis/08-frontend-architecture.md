# 08 — Frontend architecture (React / Redux / Vite)

**Phase:** 2 (2026-11-16). Source: recovered bundle `index-COO2fUbz.js` (10,547,069 B minified build — **NOT original source code**), `index.html`, `index-CvNvudRa.css`. Analysis by research agent B (Node regex scans, char offsets) + coordinator verification. Framework behavior cross-checked against Context7 (`/reactjs/react.dev`, `/reduxjs/redux-toolkit`, `/vitejs/vite`).

## 1. Framework versions

| Library | Version | Status | Evidence (char offset in bundle) |
|---|---|---|---|
| React | **18.3.1** | VERIFIED | `react_production_min.version = "18.3.1"` @15533; react-dom renderer report @236382; `reactjs.org/docs/error-decoder` URL @24560 (React 19's `react.dev` errors: 0 hits) |
| React Router | **v6 data router** | VERIFIED | `REACT_ROUTER_VERSION = "6"` @643055; `createBrowserRouter`/`RouterProvider`/`errorElement` |
| Redux Toolkit | **2.x** | VERIFIED | RTK-2 error URL `redux-toolkit.js.org/Errors?code=` @1402792; `createAsyncThunk.withTypes` @1392512 (added RTK 2.0.1) |
| redux-thunk | bundled (RTK default) | VERIFIED | `createThunkMiddleware` @1375318 |
| react-redux | v8/v9 family | STRONGLY INFERRED | `Symbol.for("react-redux-context")` + `useSyncExternalStore`; minor not pinned |
| MUI | v5/v6 family | STRONGLY INFERRED | `mui.com/production-error` @654218; `THEME_ID="$$material"`; emotion runtime styling (no Mui* classes in shipped CSS) |
| i18next + react-i18next | present | VERIFIED | `initReactI18next` @245583 |
| **connect-es + @bufbuild/protobuf** | v1-style protobuf-es | VERIFIED | `"application/grpc-web+proto"` @2367171; `MethodKind.Unary/ServerStreaming` @2360336 |
| Tauri v2 JS API | v2 | VERIFIED | `__TAURI_INTERNALS__.invoke` @5288296; `Channel`; `plugin:<name>\|<command>` |
| Froala RTE | present | VERIFIED | `FROALA_FULL_SCREEN_Z_INDEX` @2393959; 2,840 `fr-*` CSS refs |
| MathLive | **0.98.6** | VERIFIED | @7137975 |
| fabric.js | **5.3.0** | VERIFIED | @8938834 |
| WebSpellChecker | CDN service | VERIFIED | `index.html` loads `spellchecker.digiexam.com/wscservice/wscbundle/wscbundle.js` |

**Refinement of phase 1:** the frontend→backend exam API stack is specifically
**connect-es speaking the gRPC-web protocol**, while the Rust side uses **tonic
native gRPC** — both stacks coexist at different layers (review-resolved).

## 2. Build chain

- **Vite production build** — VERIFIED: `index.html` loads `/assets/index-COO2fUbz.js` +
  `/assets/index-CvNvudRa.css`, matching Vite's official `assets/[name]-[hash].[ext]`
  naming (Context7); Vite modulepreload polyfill at bundle head @3380.
- **Single-chunk output** — STRONGLY INFERRED: no `__vite__mapDeps`/preload-helper
  dynamic-import graph; one 10.5 MB entry chunk.
- **esbuild transform/minify** — VERIFIED: `__publicField` helpers, `/* @__PURE__ */`
  annotations, const-folded enums. No source maps shipped (VERIFIED absence).
- **CSS** — VERIFIED: plain global stylesheet (reset, 25 `@font-face`, Froala/MathLive
  styles, 28 media queries); MUI styling injected at runtime via emotion.

## 3. State architecture (descriptive)

- **Store:** `configureStore` with 19 `createSlice` reducers — VERIFIED slice names:
  `auth, context, form, version, error, questions, timer, answers, exam, examSettings,
  ui, accessibilityTools, proctoring, spellchecker, webView, usb, connectivity,
  notification, tts` (@5914237).
- **Typed thunks:** `createAppAsyncThunk = createAsyncThunk.withTypes()` @5599809; a 45-entry
  `ErrorSource` enum doubles as a catalogue of async flows (`exam/startOnlineExam`,
  `exam/startUsbExam`, `exam/handInOnlineExam`, `auth/*`, `answers/remoteSave*`,
  `proctoring/token`, `usb/*`, …).
- **Custom middleware — VERIFIED:**
  - `errorHandlingMiddleware` @5371529 (maps rejections to i18n messages).
  - `rustSyncMiddleware` @5906369: serialized command chain for backend sync —
    `INVOKE.EXAM_CONNECTIVITY.START/STOP`, and `setRegion` → `INVOKE.REGION.SET_SERVER_URI`;
    tracks `NativeAuthStatus = IDLE | SYNCING | DEGRADED`.
- **Exam-start flow** (consistent with phase-1 §7.4, now char-pinned @5830545):
  `startOnlineExam` thunk → `startLockdown(orgId, studentId, examId, lockdown=!openBook,
  clipboard=allowExternalPaste, isProctored=remoteProctoring)` →
  `invoke("plugin:lockdown-lite|run_all")` + event waiter on `lockdown_started` →
  gRPC `AssignmentService.startOnlineExam` (BigInt ids) → `getQuestions`/`getAnswers`;
  failure ⇒ `stopLockdown` + typed rejection. USB/offline twin: `exam/startUsbExam`
  via `plugin:io|read_to_dxe` / `write` (DXE/DXSR files).
- **Connectivity slice** @5904632: `ConnectionStatus` (server enum: UNSPECIFIED/
  INCOMPLETE/CONNECTED/REAUTHORIZATION_REQUIRED/DISCONNECTED) + `network_ping` events.

## 4. Server API surface generated into the bundle — VERIFIED

**199 generated RPCs** across services: `assignment.service.v1.AssignmentService`
(36 methods incl. `StandardLogin/LoginWithCode`, `GetActiveExams/GetExamDetails`,
`StartOnlineExam/StartUSBExam`, `GetQuestions/GetAnswers`,
`SaveTextualAnswer` (streaming) + `SaveTextualAnswerUnary`, `HandInExam`,
`GenerateProctoringToken`, `CheckProctoringLicenses`, `Get/SetConnectivityStatus`,
`RestoreExam`, `CreateLockdownSystemSnapshotUpload`), `iam.service.v1.IAMService`
(Zitadel-style), `lockdown.service.v1.LockdownService`, `lis.service.v1.LearningIntegrationService`
(LMS/Canvas), `directorysync`, `event.service.v1.EventService`, `intserver.service.v1.IntServerService/Ping`
→ see [`10-backend-protocols.md`](10-backend-protocols.md).

## 5. PLATFORM modes — VERIFIED (with esbuild folding)

`PLATFORM = { NATIVE: "native", WEB: "web", WASM: "wasm" }` @5261965. **This recovered
bundle is the NATIVE build** — the platform constant is const-folded, so platform
branches appear as literal comparisons (`"native" === PLATFORM.WASM`). Usage sites: 29
NATIVE / 13 WEB / 16 WASM.

- **NATIVE (this build):** all lockdown/IO via Tauri IPC; backend logging only when
  not WEB (`sendInfo` guard @5374686).
- **WEB:** browser exams, gRPC-web directly from the browser, no lockdown plugin.
- **WASM:** a wasm module (`initializeWasm` @5363447, `dispatchLockdownMessage`) takes
  over the lockdown dispatch role; MessagePort connections replace process IPC.
  WEB/WASM runtime behavior is reconstructed from folded code — STRONGLY INFERRED.

## 6. Version anomaly — VERIFIED, explanation STRONGLY INFERRED

The bundle contains exactly one `const version = "26.4.4"` (@10147998, coordinator
re-scan: 1 hit; `"26.4.7"`: 0 hits) while the MSI/`digiexam.exe` are **26.4.7**.
**STRONGLY INFERRED:** the frontend npm package version is managed independently of the
Tauri app version and lags the installer version. The bundle's identity is confirmed by
its embedded asset hash (see MANIFEST) — i.e., this *is* the shipped frontend of
`digiexam.exe` 26.4.7 carrying a stale package version string.

## 7. Open questions (UNVERIFIED)

- MUI/react-redux exact majors; connect-es transport construction site.
- WEB/WASM bundle builds were not analyzed (only the native build ships here).
