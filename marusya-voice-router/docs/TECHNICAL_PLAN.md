# Technical plan

## Why a normal companion app is insufficient

The target app (ChatGPT) owns its audio session and explicitly selects an Android communication device. An unrelated app cannot force another process to keep using a different output. The routing hook must execute in the ChatGPT process or be provided by a privileged system service.

## Planned implementation

- Android module package: `io.github.mikevolgov.marusyarouter`
- Target package: `com.openai.chatgpt`
- Output: selected `AudioDeviceInfo.TYPE_BLUETOOTH_A2DP`
- Input: phone built-in microphone; never request Bluetooth SCO for the microphone.
- UI: one route switch and a list of currently connected A2DP devices.
- Persistence: selected device identity and enabled state.
- Failure policy: if selected device disappears, do not leave a stale forced route; restore native routing.

## Runtime compatibility gate

Before writing the hook implementation or publishing an APK, verify the injection runtime against Android 15 and the modern Xposed API required by the target module. The previously installed LSPatch v1.2 must not be assumed compatible. Do not ask the user to uninstall ChatGPT or install a patched ChatGPT APK.

## Acceptance criteria

1. Builds reproducibly with a documented JDK/Gradle/Android SDK toolchain.
2. Installs as a separate module APK.
3. Can be enabled for ChatGPT in a compatible module manager.
4. Lists the connected VK Capsule A2DP output.
5. Routes ChatGPT Voice output to that device while retaining phone microphone capture.
6. Handles disconnect/reconnect safely.
7. Requests no contacts, call-log, or unnecessary permissions.


## Investigation update (2026-11-23)

### Candidate no-root runtime

The public fork `konglongbt/LSPatch-A15` is a source-code fork of LSPatch and its README explicitly documents both modern `libxposed` and legacy Xposed module support. Its Gradle settings include the LSPosed core as a Git submodule. This makes it a plausible runtime candidate for the modern module API used by newer modules.

However:

- The fork currently exposes no GitHub Releases (the Releases API returned an empty list).
- Its README's Android 9+ statement is not proof that this exact fork has been tested on this Realme Android 15 build.
- The fork's build depends on a pinned Git submodule and a complete Android/Gradle toolchain; it is not a ready-to-install manager APK.
- The upstream Vector framework documents Android 8.1–17 Beta compatibility, but it requires Magisk or KernelSU with Zygisk. That is not an option on this user's unrooted phone.

References:
- https://github.com/konglongbt/LSPatch-A15
- https://github.com/JingMatrix/Vector

### Critical install/data safety constraint

LSPatch patches the ChatGPT package. The patched package has a different signing identity from the Play Store installation, so Android's signature-conflict rejection is expected when attempting an in-place update. Do not uninstall the original ChatGPT app merely to test a patched build: the user's split APK backup does not contain ChatGPT's private app data or login/session state.

Before any install experiment, the implementation must prove a safe path that preserves the original installation and its data. If the only available path requires uninstalling ChatGPT, stop and explain the consequence instead of proceeding.

### Current conclusion

The technical route is plausible but not yet validated end-to-end. The next work is to inspect the LSPatch-A15 submodule/API versions and the routing module's actual source/API entry points, then build and test in CI. No APK should be offered until CI produces a real artifact and the install path is safe.


## CI build pipeline (2026-11-23)

A GitHub Actions workflow was added at `.github/workflows/marusya-voice-router.yml` on the `marusya-voice-router` branch.

- It checks out the upstream ChatGPT Audio Route source at pinned commit `6363e791ded3889b9b27b1fe6ed6057086778cad`.
- It uses JDK 17, Gradle 8.9, Android SDK 36, and Build Tools 36.1.0.
- It runs the upstream JVM tests and builds the Xposed module debug APK.
- It uploads the APK as a 14-day Actions artifact.

This pipeline intentionally builds the upstream module unchanged. A successful build proves compilation and tests only; it does not prove runtime compatibility on the Realme RMX3760 or solve the separate LSPatch signature/install conflict. Do not install a patched ChatGPT package or remove the Play Store app as part of this pipeline.

The upstream module uses the modern `libxposed` entry API and calls `AudioTrack.setPreferredDevice` for scoped Voice playback, with a separate built-in back-microphone preference. Its source is MIT-licensed and remains attributed to the original author. The public release itself excludes development-only system-server observation and legacy compatibility entries.
