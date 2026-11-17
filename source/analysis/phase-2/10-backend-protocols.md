# 10 — Backend protocols & external services

**Phase:** 2 (2026-11-16). Method-path convention verified against official tonic/gRPC documentation via Context7. All identifiers below are **observed strings** in the signed binaries/bundle; no network traffic was generated or captured for this report (all findings are static). Labels: VERIFIED / STRONGLY INFERRED / UNVERIFIED.

## 1. Transport overview

```
WebView2 frontend  ── gRPC-web (connect-es, "application/grpc-web+proto") ──► *.digiexam.com
Rust core (digiexam.exe, service.exe) ── native gRPC (tonic 0.14.5/0.13.1 over h2) ──► *.digiexam.com
digiexam.exe ── OTLP telemetry (opentelemetry-otlp, encrypted via in-house optl) ──► telemetry backend
digiexam.exe ── HTTPS (update metadata + MSI/NSIS artifacts, minisign-signed) ──► cdn.crabnebula.app
```

**Layer separation (review-resolved):** the browser-side stack is connect-es/grpc-web
(VERIFIED in bundle: `contentTypeProto = "application/grpc-web+proto"`, `ConnectError`,
`MethodKind`), the Rust-side stack is tonic (VERIFIED registry paths in *both*
digiexam.exe and service.exe). The tonic client builds request paths exactly as the
official gRPC HTTP/2 protocol requires — fully-qualified `/package.Service/Method`
(Context7/tonic docs) — matching every observed RPC string.

## 2. Observed gRPC services (Rust side, tonic) — VERIFIED

| Service / method | Evidence (digiexam.exe raw offset) |
|---|---|
| `lockdown.service.v1.LockdownService/GetLockdownConfig` | 0x01AF5F00 |
| `assignment.service.v1.AssignmentService/RefreshToken` | 0x01AF5DD8 |
| `assignment.service.v1.AssignmentService/{SetConnectivityStatus, CheckDeprecation, SaveTextualAnswer (client-streaming), CreateLockdownSystemSnapshotUpload}` | strings cluster |
| `event.service.v1.EventService/StreamClientLog` | 0x01AE5108 |
| `intserver.service.v1.IntServerService/Ping` | 0x01B2F0A8 |
| `opentelemetry.proto.collector.{logs,trace}.v1.*Service/Export` | 0x020313C8 |

**Correction (phase-3 schema reconstruction, see [`12-protobuf-schemas.md`](../phase-3/12-protobuf-schemas.md) §5):**
`LockdownStart`, `LockdownStartWithDefinitions`, `LockdownStop` and
`RollbackOperations` — listed in phase 1 as `LockdownService` methods — have **no
method-path strings in any binary**; they are **native IPC commands**
(`ExecutorIpcRequest` variants), not verified tonic RPCs. Only `GetLockdownConfig` is a
verified tonic method of `LockdownService` on the Rust side. The frontend bundle,
however, does define a `lockdown.service.v1.LockdownService` with 31 methods
(connect-es descriptors) — the client-facing API surface is larger than the Rust-side
tonic surface.

Protobuf packages: `lockdown.{parameter,opplatform,opdefinition,lifecycle}.v1` — VERIFIED
(strings_service.txt).

## 3. Observed API surface (frontend, connect-es) — VERIFIED

**199 generated RPCs** in the bundle; notable services:

- `assignment.service.v1.AssignmentService` — 36 methods: auth
  (`StandardLogin`, `LoginWithCode`, `RefreshToken`), exams (`GetActiveExams`,
  `GetExamDetails`, `StartOnlineExam`, `StartUSBExam`, `HandInExam`, `RestoreExam`),
  Q&A (`GetQuestions`, `GetAnswers`, `SaveTextualAnswer` (client-streaming —
  corrected in [`12-protobuf-schemas.md`](../phase-3/12-protobuf-schemas.md) §5),
  `SaveTextualAnswerUnary`), proctoring (`GenerateProctoringToken`,
  `CheckProctoringLicenses`), connectivity (`Get/SetConnectivityStatus`),
  snapshot upload, file up/download.
- `iam.service.v1.IAMService` — Zitadel-style IAM (`CreateZitadelAuthLoginLink`,
  IdP/policy management) — STRONGLY INFERRED vendor config of Zitadel.
- `lockdown.service.v1.LockdownService` (operation definitions/results model).
- `lis.service.v1.LearningIntegrationService` (LMS integrations incl. Canvas),
  `directorysync`, `event.service.v1.EventService`, `intserver.service.v1.IntServerService/Ping`.

## 4. Endpoints — VERIFIED strings

| Purpose | Endpoint | Evidence |
|---|---|---|
| Backend APIs | `*.digiexam.com` (incl. `auth.`, `telemetry.`) | phase-1 |
| OAuth token | `https://auth.<region>.digiexam.com/oauth/v2/token` | STRONGLY INFERRED (split format-strings; Zitadel) |
| **Update metadata** | `https://cdn.crabnebula.app/update/digiexam/digiexam/{{target}}/{{current_version}}` | VERIFIED @0x1F8DFD8 (channel `digiexam` @0x1E89528) |
| Update signature | embedded **base64-encoded minisign public key**, key-ID `B4F5E5CCB9BDE679` (`untrusted comment: minisign public key: …` / `RWR55r25…`) | VERIFIED @0x1E89558 (review correction: stored base64-wrapped, not plaintext) |
| Spellcheck | `spellchecker.digiexam.com/wscservice/wscbundle/wscbundle.js` (WebSpellChecker CDN bundle) | VERIFIED (index.html) |
| Telemetry | OTLP export via `OPTL_ENDPOINT`-configured collector; encrypted in-house (`optl` + age/hpke crates) | STRONGLY INFERRED |

## 5. Update security model (descriptive)

Official tauri-plugin-updater contract (Context7-verified): update metadata → download
over HTTPS → **minisign signature verification against the embedded public key** →
install (MSI `msiexec /i /passive /promptrestart` or NSIS `/UPDATE`) → app exit on
Windows. DigiExam's wiring matches (no custom crypto observed). The updater's privileged
install path runs through the toolbox-proxy topology — see
[`09-windows-integration.md`](09-windows-integration.md) §2.3 (host UNVERIFIED).

## 6. Open questions (UNVERIFIED)

- Complete region→host map (only `auth.`/`telemetry.` patterns observed).
- Whether the frontend grpc-web and Rust tonic stacks target identical hosts/paths.
- Actual OTLP collector endpoints (no plaintext endpoint besides the env-var indirection).
- Behavior of `digiexamtoolboxservice.exe` (referenced by fragments, not shipped here).
