# Marusya Voice Router

A dedicated Android 15 project for the specific setup:

- Realme RMX3760 / Android 15
- ChatGPT Voice playback to a connected VK Capsule / Marusya Bluetooth A2DP speaker
- Voice input from the phone's built-in microphone

## Important technical constraint

A normal Android app cannot reliably override another app's active communication route. ChatGPT selects its own communication device, so a separate controller app alone cannot solve this. The actual routing component must run inside the ChatGPT process through a compatible Xposed/LSPosed-style runtime (or another privileged system integration).

This project is therefore being developed as a small, dedicated routing module plus a simple configuration UI. It will not modify the CSQTT app or require uninstalling the original ChatGPT app.

## Target behavior

1. Detect connected Bluetooth A2DP outputs.
2. Let the user choose the Marusya speaker.
3. Keep microphone capture on the phone.
4. Override ChatGPT Voice playback to the selected A2DP output.
5. Fall back safely to Android's native route if the speaker disconnects.
6. Avoid requesting contacts, call-log, or unrelated permissions.

## Build status

This branch currently contains the project specification only. No installable APK has been built yet. The next engineering gate is verifying a no-root Android 15 injection runtime that can load the module. Do not install an APK or remove the original ChatGPT app as part of this work.

## Device acceptance test

- YouTube audio continues to work on the speaker.
- ChatGPT Voice audio comes from the speaker.
- Your voice is captured by the phone microphone.
- Disconnecting the speaker restores normal phone audio without a crash.
