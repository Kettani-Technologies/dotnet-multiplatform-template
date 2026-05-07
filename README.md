# dotnet-multiplatform-template

A minimal `.NET 10` + Uno Platform multi-target reference and starter template
for the legion's mobile / cross-platform development. Demonstrates a single
codebase compiling to **six platforms** with a working GitHub Actions matrix
that builds them all in parallel from any push.

## Why this exists

This is the canonical answer the legion landed on for "how do we build .NET
mobile and cross-platform apps in 2026 from a Linux wing":

- **Single codebase** for Android, iOS, Mac Catalyst, Windows (WinAppSDK),
  WebAssembly, and Linux desktop.
- **Agent-friendly** — pure CLI builds, no IDE designer dependency, fast
  inner loop on Linux/WASM, slow targets (iOS/macOS/Windows) delegated to
  GitHub Actions runners.
- **One workflow file** at `.github/workflows/build.yml` covers all six
  targets with proper caching and artifact upload. Fork or clone this repo
  to inherit the working pipeline.

## Targets

The single project multi-targets six TFMs. Five build clean today; iOS and
Mac Catalyst are gated by a transient `Xcode 26.4 vs 26.3` mismatch on
GitHub-hosted macOS runners (resolves automatically when GH ships Xcode 26.4).

| TFM                           | Built on        | Output           | Status |
| ----------------------------- | --------------- | ---------------- | ------ |
| `net10.0-desktop`             | `ubuntu-latest` | Linux Skia app   | ✓      |
| `net10.0-browserwasm`         | `ubuntu-latest` | WebAssembly site | ✓      |
| `net10.0-android`             | `ubuntu-latest` | APK + AAB        | ✓      |
| `net10.0-windows10.0.19041`   | `windows-latest`| MSIX             | ✓      |
| `net10.0-ios`                 | `macos-15`      | unsigned `.app`  | (Xcode pin) |
| `net10.0-maccatalyst`         | `macos-15`      | unsigned `.app`  | (Xcode pin) |

## Two ways to use this template

### Option 1 — Fork or clone

```bash
git clone https://github.com/Kettani-Technologies/dotnet-multiplatform-template MyApp
cd MyApp
# Rename ElysiumUnoCanary → MyApp throughout
# Update ApplicationId, ApplicationTitle in csproj
git remote set-url origin <your-new-repo>
git push -u origin main
```

### Option 2 — Install as a `dotnet new` template

`dotnet new install` doesn't accept git URLs directly, so clone first then
install from the local path:

```bash
git clone https://github.com/Kettani-Technologies/dotnet-multiplatform-template /tmp/dmt
dotnet new install /tmp/dmt
dotnet new dotnet-multiplatform -n MyApp
rm -rf /tmp/dmt   # template now registered with the .NET CLI; clone can go
```

The template parameterizes the project name automatically: file names,
directory names, namespaces, and `ApplicationId` all derive from `-n`.
Recommended for new projects.

## Local repro on a Linux dev machine

Assuming `.NET 10 SDK`, `JDK 21`, and the Android SDK are installed (the
legion's wing hosts pre-provision all three):

```bash
dotnet new install Uno.Templates                       # once per user

dotnet workload restore                                # installs TFM-implied workloads
dotnet workload install wasm-tools wasm-experimental   # NuGet-implied, restore misses these

dotnet build -c Release -f net10.0-desktop
dotnet build -c Release -f net10.0-browserwasm
dotnet build -c Release -f net10.0-android \
  -p:AndroidSdkDirectory=$ANDROID_SDK_ROOT
```

`net10.0-ios`, `net10.0-maccatalyst`, and `net10.0-windows10.0.19041` require
their respective host operating systems — delegated to GitHub Actions.

## Workflow design notes

`.github/workflows/build.yml` runs on push to `main`, pull requests, and
manual `workflow_dispatch`. Three lessons baked into it:

1. **`dotnet workload restore` instead of `install`** — Uno's csproj declares
   all six TFMs in `<TargetFrameworks>`, so building any one TFM still
   requires every TFM's workload. `restore` reads the project and installs
   what's needed in one step.
2. **Explicit `wasm-tools` install** anyway — Uno pulls wasm requirements via
   NuGet (`Uno.Wasm.Bootstrap`), not via TFM-implied workloads, so `restore`
   misses it.
3. **`xcode-select` to newest preinstalled Xcode** on `macos-15`, since the
   default Xcode is too old for `.NET 10` Apple SDKs.

## What this template does **not** do

- No code signing — outputs are unsigned and not installable on locked devices.
- No deploy to Play Store / App Store / Microsoft Store.
- No real Android emulator / iOS simulator runs.
- No UI / integration tests.

These are layered on top once signing certificates, store credentials, and
test harnesses are configured as GitHub Actions secrets.
