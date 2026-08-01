# AGENTS.md — App Manager

## Project Overview

**App Manager** is a full-featured, open-source (GPL-3.0-or-later) Android application for managing installed apps, system operations, permissions, backups, and device internals. It combines the features of 5–6 apps into one, with support for root and ADB modes.

- **License:** GPL-3.0-or-later (individual files may be dual-licensed, check SPDX headers)
- **Package:** `io.github.muntashirakon.AppManager`
- **Min SDK:** 21 (Android 5.0) | **Target/Compile SDK:** 36
- **Build tools:** 36.1.0 | **AGP:** 8.13.2 | **Gradle Wrapper:** provided
- **Primary language:** Java (source compatibility Java 8), with JNI (C/C++ via CMake)
- **Testing:** JUnit 4 + Robolectric
- **Repository mirrors:** Codeberg, GitLab, Riseup, sourcehut (PRs accepted only on GitHub)

---

## Useful Commands

### Setup

```bash
# Clone with submodules (required)
git clone --recurse-submodules https://github.com/MuntashirAkon/AppManager.git

# Open in Android Studio / IntelliJ IDEA — sync happens automatically
```

### Build

```bash
# Build debug universal APK
./gradlew packageDebugUniversalApk

# Run lint checks
./gradlew lint

# Run unit tests
./gradlew test

# Build release universal APK
./gradlew packageReleaseUniversalApk

# Create bundled app (APKS) from AAB (requires bundletool on PATH)
./scripts/aab_to_apks.sh release   # or: debug
./scripts/aab_to_apks.sh release false   # non-interactive (unsigned)

# Build docs (requires pandoc + latex)
./scripts/make_docs.sh
```

### Development Helpers (scripts/)

| Script | Purpose |
|--------|---------|
| `scripts/make_docs.sh` | Generate localized docs into the docs module resources |
| `scripts/push_to_mirrors.sh` | Push master to all mirror remotes |
| `scripts/fix_strings.php` | Fix or validate string resources |
| `scripts/make_suggestions.php` | Generate permission/app-op suggestions |
| `scripts/make_debloat_list.php` | Generate debloat JSON from community contributions |
| `scripts/update_libraries.php` | Update library signature definitions |
| `scripts/docs.php` | Doc utility script |

### CI/CD (GitHub Actions)

- **Lint:** `lint.yml` — runs `./gradlew lint` on push/PR to master and `AppManager-*` branches
- **Tests:** `tests.yml` — runs `./gradlew test` on push/PR
- **CodeQL:** `codeql.yml` — weekly Java security analysis

---

## Technologies

### Languages & Runtimes
- **Java** (source compatibility: Java 8; build with JDK 17+), with core library desugaring
- **C/C++** via CMake for native code (`app/src/main/cpp/`)
- **AIDL** for IPC interfaces
- **PHP** (utility scripts: debloat list generation, string fixing, documentation)
- **Bash** (CI/build scripts, mirror pushing)

### Build System
- **Gradle** with Groovy DSL (settings.gradle, build.gradle, versions.gradle)
- **Android Gradle Plugin (AGP)** 8.13.2
- **HiddenApiRefinePlugin** 4.0.0 for accessing hidden Android APIs

### Module Structure (`settings.gradle`)

| Module | Purpose |
|--------|---------|
| `app` | Main application |
| `docs` | In-app documentation module |
| `hiddenapi` | Compile-only stubs for hidden Android APIs |
| `libcore:compat` | Compatibility utilities |
| `libcore:io` | I/O utilities |
| `libcore:ui` | Shared UI components |
| `libopenpgp` | OpenPGP encryption support |
| `libserver` | Server module library |
| `server` | Background server process (ADB/root operations) |

### Key Architecture Decisions
- **Single `Application` subclass:** `AppManager.java` — handles Security providers, HiddenApiBypass, Shell config, and crash handler
- **Feature-based packaging:** each feature has its own package under `io.github.muntashirakon.AppManager.*` (e.g., `backup/`, `db/`, `debloat/`, `logcat/`, `profiles/`, etc.)
- **ADB/Root operations** are handled through the `server` module, with `libadb-android` for ADB communication and `libsu` for root
- **Room Database** v2.7.2 for persistence with incremental schema generation
- **Hidden API access** via `LSPosed/AndroidHiddenApiBypass` (exempts all classes via `"L"`)

### Key Dependencies

| Area | Libraries |
|------|-----------|
| **UI** | Material 3 (1.13.0), AppCompat (1.7.1), AndroidX Core (1.17.0) |
| **APK Handling** | apksig-android (4.4.0), ARSCLib, unapkm-android, smali/baksmali (3.0.9) |
| **APK Reverse Engineering** | jadx-android (1.4.7) |
| **Security/Crypto** | Bouncy Castle (1.83), sun-security-android (1.1), OpenKeychain (libopenpgp) |
| **Compression** | zstd-jni (1.5.7-7) |
| **JSON** | Gson (2.13.2) |
| **File Type Detection** | simplemagic (1.17) |
| **Code Editor** | sora-editor (0.22.2) |
| **Testing** | JUnit 4 (4.13.2), Robolectric(4.16.1) |

### Native Code
- Located at `app/src/main/cpp/` for CMake
- Supports ABI splits: `armeabi-v7a`, `arm64-v8a`, `x86`, `x86_64`
- Universal APK build enabled

---

## Best Practices & Guidelines

### Licensing & SPDX
- **Every file** must include an SPDX-License-Identifier header
- **Default license:** `GPL-3.0-or-later` (the project wide license)
- **Files with other licenses:** Add `GPL-3.0-or-later` with an `AND` conjunction: `SPDX-License-Identifier: Apache-2.0 AND GPL-3.0-or-later`
- **NO `@author` tag** — it is considered bad practice
- For copied files/classes from another person: add a copyright statement like `// Copyright 2004 Linus Torvalds`

### git Commit Conventions
- **Sign off** every commit with `--signoff` or `Signed-off-by: Name <email>`
- Follow Linux kernel commit message conventions
- Use real name and real email (legal requirement for license changes)
- **NO AI-Generated Code:** the project explicitly rejects all AI/LLM-generated contributions

### Code Quality
- Build passes lint (`./gradlew lint`)
- All tests pass (`./gradlew test`)
- Minimal dependencies — prefer existing APIs over new libraries
- Use `@Keep` annotations on methods called via reflection
- Use `@NonNull`/`@Nullable` annotations for null safety
- Use `BuildConfig.DEBUG` for debug-only behavior (not manual flags)
- Desugaring enabled via `coreLibraryDesugaringEnabled true` — use Java 8+ APIs freely

### Architecture Rules
- New features go into their own package under `io.github.muntashirakon.AppManager`
- Shared UI components belong to `libcore:ui`
- I/O utilities belong to `libcore:io`
- Compatibility utilities belong to `libcore:compat`
- Hidden API stubs belong to the `hiddenapi` module (compile-only)
- The `server` module handles ADB and root-privileged operations
- Use `StaticDataset.java` for caching frequently-accessed data

### Security
- No hardcoded secrets in production code (debug only)
- `*.jks` files in `.gitignore` (except `app/dev_key_store.jks` for debug signing)
- Hidden ApiBypass is applied in `attachBaseContext`
- Security providers are configured in `AppManager.onCreate()`
- Cryptographic operations should use Bouncy Castle or JavaKeyStoreProvider
- SLF4J is excluded from dependencies (use Android logging)

### Resource Handling
- String resources in `app/src/main/res/` — translatable via Weblate
- Use `fastlane/` metadata for app store listings
- Documentation lives in `docs/raw/` — mark in multiple languages (`en/`, `de/`, `es/`, `ja/`, `ru/`, `zh-rCN/`)

### Testing
- Unit tests under `app/src/test/`
- Test runner: JUnit 4 (NOT JUnit 5)
- Robolectric for Android framework tests
- Test schemas at `app/schemas/` (Room schema export)
- CI runs `./gradlew test` — make sure they pass before pushing

### Versioning
- Version code declared in `app/build.gradle` (`versionCode`)
- Version name declared there too (`versionName`)
- Dependency versions centralized in `versions.gradle` — always add new deps there

---

## Environment Setup

### Prerequisites
- JDK 17 or higher (CI uses Zulu JDK 21)
- Android Studio or IntelliJ IDEA
- 8 GB RAM minimum, 20 GB disk space
- Windows/Linux/macOS (WSL supported)
- Active network connection (initial sync can download ~20 GB)

### Getting Started
1. Clone with `--recurse-submodules`: `git clone --recurse-submodules https://github.com/MuntashirAkon/AppManager.git`
2. Open the project in Android Studio — Gradle syncs automatically
3. Run `./gradlew packageDebugUniversalApk` to build

### Code Reviews
- All commits are thoroughly examined before merge
- Pull requests are accepted only on GitHub; mirrors are receive-only
- Contributions are licensed under GPL-3.0-or-later by default
- Sign-off is mandatory (`Signed-off-by:` in commit message)

### Maintaining Backward Compatibility
- Min SDK is 21 — ensure APIs used are available on Android 5.0+
- Use `Build.VERSION.SDK_INT` checks for version-specific code
- Use `Utils.isAtLeast*()` helpers when available
- Use resource configuration qualifiers instead of runtime checks where applicable

### Release Process
1. Update `versionCode` and `versionName` in `app/build.gradle`
2. Run `./gradlew lint && ./gradlew test` and ensure they pass
3. Build: `./gradlew packageReleaseUniversalApk` or `./scripts/aab_to_apks.sh release`
4. Sign with release keystore 
5. Push to mirrors with `./scripts/push_to_mirrors.sh`

---

## Translation

- App strings: https://hosted.weblate.org/engage/app-manager/
- Documentation: https://hosted.weblate.org/projects/app-manager/docs/
- All translations are crowd-sourced; review carefully for context
- Use `scripts/fix_strings.php` to validate string resources after editing