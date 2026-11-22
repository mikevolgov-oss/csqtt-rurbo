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
