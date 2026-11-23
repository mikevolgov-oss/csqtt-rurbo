# Loader compatibility findings

Updated: 2026-11-23

## Current module

The built `xposed-debug.apk` is the ChatGPT Audio Route module, built from upstream commit `6363e791ded3889b9b27b1fe6ed6057086778cad`. It uses the modern `libxposed` API. The GitHub Actions artifact is a module APK, not a ChatGPT APK or a loader.

## Candidates

### LSPatch-A15

Repository: https://github.com/konglongbt/LSPatch-A15

The project README describes a rootless per-app patcher and says it supports both modern `libxposed` modules and legacy Xposed modules. It is the closest loader candidate for this module. The repository currently has no published GitHub release assets, so a locally installable manager/patcher build has not been verified.

Important blocker: patching the installed official ChatGPT APK changes its signing certificate. Android Package Manager will reject it as an update over the official app. Do not uninstall the official ChatGPT app or install a patched replacement without an explicit, informed user decision and a tested recovery plan.

### ZenPatch

Repository: https://github.com/saintxseiyax/ZenPatch

Do not use the currently published alpha for this project:
- Its own security audit says the DEX loader in alpha01 is a stub.
- Its runtime module loader imports the legacy `de.robv.android.xposed` API, whereas ChatGPT Audio Route is a modern `libxposed` module.
- Its release is a prerelease with no downloadable APK assets.
- Its signature-spoofing feature is an in-process PackageManager hook; it does not change Android's install-time signing-certificate check.

## Next non-destructive avenue to investigate

Investigate a separately installed ChatGPT clone with a changed package name, then patch only that clone with a compatible LSPatch-A15 build and adapt the module scope to the clone package. This is only a research candidate: package renaming may break ChatGPT login, OAuth/deep links, or app integrity checks. Do not tell the user it works until tested. Keep the official ChatGPT installation and CSQTT untouched.
