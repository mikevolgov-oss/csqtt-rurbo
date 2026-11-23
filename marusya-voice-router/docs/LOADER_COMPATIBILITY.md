# Loader compatibility findings

Updated: 2026-11-23

## Current module

The built `xposed-debug.apk` is the ChatGPT Audio Route module, built from upstream commit `6363e791ded3889b9b27b1fe6ed6057086778cad`. It uses the modern `libxposed` API. The GitHub Actions artifact is a module APK, not a ChatGPT APK or a loader.

## Candidates

### LSPatch-A15

Repository: https://github.com/konglongbt/LSPatch-A15

The project README describes a rootless per-app patcher and says it supports both modern `libxposed` modules and legacy Xposed modules. Its CI build completed successfully on 2026-11-23. The artifact contains a manager APK and patcher JAR, not a patched ChatGPT APK.

Source inspection confirmed that the patcher preserves the target package name. It does not provide a package-ID rename option. Patching the installed official ChatGPT APK changes its signing certificate, so Android Package Manager rejects it as an update over the official installation. Do not uninstall the official ChatGPT app or install a patched replacement without a tested recovery plan.

### APK Cloner (research candidate only)

Repository: https://github.com/HaveAGo1/Apk-Cloner

The 0.5.0 source (last commit 2026-04-13) contains a substantially more complete clone pipeline than a manifest-only renamer:
- rewrites the binary manifest and resources table;
- can rewrite DEX package strings/types (Deep Clone), or inject a package-name shim;
- rewrites provider authorities and signs the resulting clone;
- can preserve split-APK bundles as `.apkm`;
- optionally hooks `PackageManager.getPackageInfo()` inside the clone to report the original signing certificates.

This makes it a technically plausible way to create a second package without uninstalling the Play Store ChatGPT. It is not yet a safe/validated solution:
- signature spoofing is an in-process PackageManager hook only; it does not restore the original APK signing certificate, satisfy install-time signature checks, or bypass server-side app-integrity checks;
- the README explicitly warns that apps with certificate pinning or runtime package-integrity checks may fail;
- package rename/deep rewrite may break ChatGPT OAuth callbacks, Android App Links, account login, or future updates;
- the repository has no GitHub Releases, no root LICENSE file was present in the inspected tree, and its latest commit is unsigned;
- its Android app targets SDK 28 and compiles against SDK 34; no Android 15 / ChatGPT compatibility test is published.

Do not install an APK built from this source on the user's phone until provenance, licensing, build reproducibility, and the actual ChatGPT clone have been reviewed. Do not enable signature spoofing as a presumed fix for OAuth or server integrity.

Source inspection:
- README and changelog: https://github.com/HaveAGo1/Apk-Cloner
- latest source commit: https://github.com/HaveAGo1/Apk-Cloner/commit/d1359f87495ff139461a8d357500ff1f2ab06dad
- `CloneEngine.kt`, `ManifestPatcher.kt`, `DexPatcher.kt`, and `SignatureCapture.kt` on the repository's `main` branch.

### ZenPatch

Repository: https://github.com/saintxseiyax/ZenPatch

Do not use the currently published alpha for this project:
- Its own security audit says the DEX loader in alpha01 is a stub.
- Its runtime module loader imports the legacy `de.robv.android.xposed` API, whereas ChatGPT Audio Route is a modern `libxposed` module.
- Its release is a prerelease with no downloadable APK assets.
- Its signature-spoofing feature is an in-process PackageManager hook; it does not change Android's install-time signing-certificate check.

## New build: clone-scoped audio module

A dedicated workflow now builds a second variant of the audio module for the planned independent clone package:

- Target package: `com.openai.chatgpt.marusya`
- Workflow: `.github/workflows/marusya-clone-module.yml`
- Upstream source is pinned to `6363e791ded3889b9b27b1fe6ed6057086778cad`.
- The workflow rewrites the module's runtime package constant, Xposed scope list, and Android manifest scope/query, then runs JVM tests and builds the Xposed APK.
- The build verifies that the packaged `META-INF/xposed/scope.list` contains the clone package.

To use this module, any separately cloned ChatGPT app must be created with exactly the package ID `com.openai.chatgpt.marusya`; the APK Cloner's automatic package-name suggestion is not sufficient. This workflow does not create the ChatGPT clone and does not prove that ChatGPT login works under the new package.

## Remaining end-to-end blockers

The clone must still be generated from the user's installed base + split APKs, installed alongside the official app, and successfully signed into. Then the clone must be patched by a compatible LSPatch-A15 build in a way that preserves all required split APKs. Only after those steps can the audio module be loaded and the Realme/VK Capsule route tested. The original ChatGPT app and CSQTT Turbo remain untouched.

## Remaining end-to-end blockers

Even if a separate ChatGPT clone can be produced, it still needs:
1. a working clone install that retains ChatGPT sign-in and network access;
2. a LSPatch-A15 path that correctly handles the ChatGPT base APK plus all required split APKs;
3. a module build whose scope and manifest target the clone package ID;
4. a verified route test on the Realme RMX3760 / Android 15: Voice output to VK Capsule A2DP while capture stays on the phone microphone.

No published evidence currently verifies these four points together. The clone route is a research lead, not a finished APK.

## Safety constraints

Keep the official ChatGPT installation and its private app data intact. Do not ask the user to uninstall it. Do not alter the working CSQTT Turbo code. A successful CI build alone is not proof of installation, login, or Bluetooth routing on the target phone.
