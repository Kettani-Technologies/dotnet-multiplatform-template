# elysium-uno-canary

A minimal `.NET 10` + Uno Platform multi-target canary used to verify that build
pipelines stay green across all reachable platforms from Wing Elysium.

## Targets

The single project multi-targets four TFMs:

| TFM                        | Built on        | Output           |
| -------------------------- | --------------- | ---------------- |
| `net10.0-desktop`          | `ubuntu-latest` | Linux Skia app   |
| `net10.0-browserwasm`      | `ubuntu-latest` | WebAssembly site |
| `net10.0-android`          | `ubuntu-latest` | APK + AAB        |
| `net10.0-ios`              | `macos-14`      | unsigned `.app`  |

Mac Catalyst and Windows (WinAppSDK) are not in the default Uno template's
`<TargetFrameworks>`; add them and a matching workflow entry when needed.

## How the workflow runs

`.github/workflows/build.yml` triggers on:

- push to `main`
- any pull request
- manual dispatch from the **Actions** tab

A matrix runs each TFM as an independent job in parallel. NuGet packages are
cached per OS + TFM. Build artifacts (APK, AAB, WASM `wwwroot`, etc.) are
uploaded for 7 days so they can be downloaded from the run summary page.

## What this canary does **not** do

- No code signing — outputs are unsigned and not installable on locked devices.
- No deploy to Play Store / App Store / Microsoft Store.
- No real Android emulator / iOS simulator runs.
- No UI / integration tests.

These can be layered on top once we have signing certificates, store
credentials, and a UI test harness configured as GH Actions secrets.

## Local repro

On Wing Elysium (or any Linux box with the matching toolchain):

```bash
dotnet new install Uno.Templates                # once per user
dotnet build ElysiumUnoCanary/ElysiumUnoCanary.csproj -c Release -f net10.0-desktop
dotnet build ElysiumUnoCanary/ElysiumUnoCanary.csproj -c Release -f net10.0-android \
  -p:AndroidSdkDirectory=$ANDROID_SDK_ROOT
dotnet build ElysiumUnoCanary/ElysiumUnoCanary.csproj -c Release -f net10.0-browserwasm
```

`net10.0-ios` requires macOS with Xcode — delegated to GH Actions.
