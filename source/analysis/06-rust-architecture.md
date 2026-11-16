# 06 — Rust architecture

**Phase:** 2 (2026-11-16) · **Method:** read-only string/offset analysis by research agent A, cross-checked by review agent (see [`11-tooling-and-methodology.md`](11-tooling-and-methodology.md)). Labels: **VERIFIED** (byte-exact at cited offset), **STRONGLY INFERRED**, **UNVERIFIED**. Binary hashes in [`01-executable-provenance.md`](01-executable-provenance.md). No binary was executed or modified.

## 1. Workspace topology — VERIFIED

Panic-path strings reveal a **Cargo workspace monorepo rooted at `native\`**:

```
native\
├── apps\
│   ├── digiexam\                  → digiexam.exe (crate app_lib: main.rs, lib.rs)
│   ├── lockdown-windows-service\  → service.exe
│   └── toolbox-ipc-proxy\         → toolbox-ipc-proxy.exe
└── libs\
    ├── lockdown-core\        (operation engine: handler\mod.rs, handler_factory.rs,
    │                          recovery_log.rs, notify_handler.rs, handlers\windows\*)
    ├── lockdown-ipc\         (framing.rs, client.rs, server.rs, codesign.rs)
    ├── lockdown-windows\     (registry.rs, service.rs, process\signing.rs,
    │                          audio\session_manager.rs, vm_detection\x86_64\timing.rs)
    ├── lockdown-shared\      (clipboard\clipboard_blocker.rs)
    ├── protocol-client\      (generated tonic clients: *.service.v1.tonic.rs)
    ├── grpc\                 (network_manager\*, ping_service.rs)
    ├── optl\                 (in-house telemetry: backend_native\encryption.rs,
    │                          optl_builder.rs, legacy\batch_stream.rs, reload_builder.rs)
    ├── digiexam-models\      (dxe.rs — the .dxe exam-package format)
    ├── anti-dll-injection\   (client)
    └── diagnostics\
```

Shared `lockdown-core` + `lockdown-windows` compile into **both** `digiexam.exe` and
`service.exe` — one operation engine, two hosts.

## 2. Core dependency stack — VERIFIED (registry-path panic strings)

Unique crate-version pairs parsed from `.cargo\registry\src\index.crates.io-...\` paths:
**188 in `digiexam.exe`, 130 in `service.exe`, 10 in `toolbox-ipc-proxy.exe`.**

| Family | Crates (version VERIFIED unless noted) |
|---|---|
| Tauri stack | `tauri-2.10.3` @0x01AFF688, `tauri-runtime-wry-2.10.1` @0x01B04D7A, `webview2-com-0.38.2` @0x02099138, `tao 0.34.8`, `muda-0.17.2`, `dpi 0.1.2`; **`wry` via git checkout** `wry-b22f37efb3452b8e\7104ed2` @0x01B08CF8 (not crates.io — review-corrected) |
| Official plugins | `updater-2.10.1`, `deep-link-2.4.7`, `store-2.4.2`, **`global-shortcut-2.3.2` @0x01B1207E**, **`os-2.3.2` @0x01B0DA99** (both newly identified in phase 2) |
| First-party plugins | `lockdown-lite`, `webview-manager`, `io`, `battery`, `keyboard-layout` |
| Async | `tokio-1.54.1`, `tokio-util 0.7.18`, `mio 1.2.0`, `futures-util 0.3.32`, `tower 0.5.3`, `backon 1.6.0` (retry), `parking_lot`, `dashmap`, `arc-swap` |
| HTTP/TLS | `hyper-1.9.0`, `h2-0.4.16`, `http-1.4.0`; **two `reqwest` copies: 0.13.4** @0x01AF5A6A **and 0.12.28** @0x0205ADD8 (dual-version confirmed in *both* client and service); TLS via **rustls 0.23.46** + `aws-lc-rs-1.18.1` + `ring 0.17.14` + `rustls-platform-verifier 0.6.2` **and** native-TLS/`schannel 0.1.29` stack compiled in parallel |
| gRPC/protobuf | **two `tonic` copies: 0.14.5** @0x01AE4FF6 **and 0.13.1** @0x02045741 (both binaries); `tonic-prost-0.14.5`; prost confirmed by codec path `tonic-0.13.1\src\codec\prost.rs` (no standalone prost registry path) |
| DNS | `hickory-resolver-0.26.0` |
| Serialization | `serde 1.0.228`, `serde_json-1.0.149`, `serde_with 3.21.0` (service), `base64`, `uuid 1.23.0`, `chrono 0.4.44`, `jiff`, `time` |
| Observability | `tracing-*`, `opentelemetry 0.30/0.31` + `opentelemetry-otlp-0.30.0` (OTLP export paths verified), in-house `optl` layer with **encrypted backend: `age-0.11.2`, `hpke-0.13.0`, `scrypt`, `curve25519-dalek`, `aes`** — purpose STRONGLY INFERRED from `OPTL_ENDPOINT` env var + wiring (`apps\digiexam\src\service\optl.rs`); "optl" expansion **UNVERIFIED** |
| Windows | `windows-0.62.2` (+0.58.0/0.61.3), `windows-service-0.8.0` (service only), `wmi-0.17.3`/`0.18.4`, `winreg`, `schannel`, `sysinfo-0.37.2`, `ipconfig`, `kill_tree 0.2.4` |
| Misc | `zip 4.6.1`+`9.0.5`, `minisign-verify 0.2.5` (updater signatures), `i18n-embed`/`fluent-*`, `moka`, `regex`, `url`, `cookie`, `cfb` |

## 3. Binaries → crates mapping — VERIFIED

| Binary | Composition |
|---|---|
| `digiexam.exe` | `app_lib` (Tauri 2.10.3 app): thin `main.rs`, `lib.rs` builds the runtime; **commands layer** `command\{exam, auth, region, sec, app, logger, updater}.rs`; **stateful services** `service\{connectivity, optl, updater\updater_service}`; the five first-party plugins; embeds `lockdown-core` (client-side rollback: `rollback_from_recovery_log`) |
| `service.exe` | Headless (zero Tauri strings — consistent with phase 1/2 marker counts): `windows-service 0.8.0` SCM integration, tokio named-pipe server, same `lockdown-core` engine, gRPC client, OTLP export, unique `sysinfo-collector` crate (snapshot upload: `CreateLockdownSystemSnapshotUpload`) |
| `toolbox-ipc-proxy.exe` | Thin stdio↔named-pipe forwarder using only `lockdown-ipc` (`client.rs`, `framing.rs`); no SCM, no advapi32 |

## 4. Build provenance anomaly — VERIFIED (notable)

- `digiexam.exe` and `service.exe` panic paths: `C:\Users\runneradmin\.cargo\...` (CI build).
- `toolbox-ipc-proxy.exe` panic paths: `C:\Users\AndreVennberg\RustroverProjects\frontend\native\...`
  + ships PDB name `toolbox_ipc_proxy.pdb` + **older dependency versions** (e.g. `tokio 1.48.0`, `serde_json 1.0.145` vs 1.54.1/1.0.149).

**STRONGLY INFERRED:** the proxy is a separately built, version-skewed component
included in one shipped release — relevant to supply-chain review of the product.

## 5. Interfaces to adjacent components

- **WebView2** → see [`07-tauri-ipc.md`](07-tauri-ipc.md) (Tauri IPC) and [`09-windows-integration.md`](09-windows-integration.md).
- **Named-pipe IPC** (`lockdown-ipc` + `codesign.rs` peer validation) → [`09-windows-integration.md`](09-windows-integration.md) §2.
- **gRPC** → [`10-backend-protocols.md`](10-backend-protocols.md).

## 6. Open questions (UNVERIFIED)

- Full expansion/purpose of the `optl` telemetry layer (encrypted local files).
- Why two versions each of tonic/reqwest (likely transitive pinning — one copy per dependency subtree).
- Whether `digiexam.exe` also self-hosts the toolbox pipe (see [`09-windows-integration.md`](09-windows-integration.md) §2.3).
