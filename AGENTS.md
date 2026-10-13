# AGENTS.md — Network Monitor Mobile App

## Project Overview

**Goal:** A cross-platform native mobile app (iOS + Android) to monitor a local
network using ping, nmap-style scanning, nslookup/DNS, and other tools — with a
single shared codebase that does **not** depend on the host system at runtime.

**Framework:** Flutter + Dart. Flutter compiles to native ARM/x86 machine code
(AOT) and bundles its own engine, so the app ships independently of any
system-level tooling.

## Directory Layout (scaffolding plan)

```
network-monitor/
├── lib/
│   ├── main.dart              # App entry point + theme
│   ├── app.dart               # MaterialApp + routes
│   ├── features/
│   │   ├── ping/              # ICMP ping feature
│   │   ├── scan/              # Nmap-style network scan
│   │   ├── dns/               # nslookup / DNS lookup
│   │   └── dashboard/         # Summary screen
│   ├── core/
│   │   ├── models/            # Shared data models
│   │   └── utils/             # Helpers (formatting, permissions)
│   └── l10n/                  # Localization (arb files)
├── test/
│   ├── unit/                  # Logic tests
│   └── widget/                # Widget tests
├── integration_test/          # End-to-end (device/emulator)
├── pubspec.yaml
├── AGENTS.md
└── README.md
```

## Development Commands

| Task              | Command                      |
|-------------------|------------------------------|
| Install deps      | `flutter pub get`            |
| Run on device     | `flutter run`                |
| Run on web        | `flutter run -d chrome`      |
| Run unit tests    | `flutter test test/unit`     |
| Run widget tests  | `flutter test test/widget`   |
| Run all tests     | `flutter test`               |
| Format code       | `dart format lib/ test/`     |
| Analyze / lint    | `dart analyze`               |
| Build APK         | `flutter build apk --release`|
| Build iOS (macOS) | `flutter build ios --release`|
| Clean build       | `flutter clean && flutter pub get` |

## Core Dependencies (planned)

| Package               | Purpose                                   |
|-----------------------|-------------------------------------------|
| `ping_discover`       | Ping + network discovery (pure Dart)      |
| `dns_lookup` / built-in | DNS / nslookup functionality           |
| `permission_handler`  | Request network & location permissions    |
| `network_info_plus`   | Get current WiFi gateway/subnet           |
| `provider` or `riverpod` | State management                    |
| `fl_chart`            | Dashboard charts (latency, history)       |

> **Note on nmap:** True nmap cannot run on iOS without jailbreaking.
> The "scan" feature will implement a **pure-Dart TCP SYN / ARP-style sweep**
> of the local subnet — scanning common ports (22, 80, 443, 53, etc.) — which
> gives equivalent host-discovery without external binaries.

## Architecture Notes

- **No system shell.** All network operations run in Dart or call into Flutter's
  platform channels with explicit Kotlin/Swift fallbacks. The app works on a
  clean phone with zero setup.
- **Permissions:** Android needs `ACCESS_FINE_LOCATION` + `ACCESS_WIFI_STATE`
  to get gateway IP. iOS shows a system dialog; no extra entitlement required for
  local ping in recent versions. Request at runtime via `permission_handler`.
- **State management:** Use `riverpod` — compile-time safety + testable.
- **Testing:** Mock network calls with `mockito`. Unit-test scan logic in pure
  Dart. Run widget tests for UI components.
- **Build flavor:** Use "debug" for development, "release" for store builds.
  No "profile" needed for this simple tool.

## Testing Conventions

- **Unit tests** live in `test/unit/` — test pure Dart logic (models, formatters,
  scan algorithms) with no Flutter bindings.
- **Widget tests** in `test/widget/` — render individual screens with fake data.
- **Integration tests** in `integration_test/` — run on a real device or emulator.
  Requires WiFi connected to a local network.
  ```
  flutter drive --target=integration_test/app_test.dart
  ```
- **Coverage:** `flutter test --coverage` → check `coverage/lcov.info`.
  Aim for >70% coverage on core scan/ping logic.

## Gotchas

- On Android 13+ (API 33), `NEARBY_DEVICES` permission is also needed for
  Bluetooth-based discovery if added later.
- iOS Simulator cannot do real ping (no ICMP socket). Use a physical device or
  test the scan feature on web (Chrome) for development.
- Hot reload works, but changes to native `android/` or `ios/` folders require
  a full stop/start.
- Never commit `pubspec.lock` for library projects, but DO commit it for apps
  (this is an app → commit it).
