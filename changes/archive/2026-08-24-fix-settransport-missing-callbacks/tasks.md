## 1. Fix the early-return path in setTransport()

- [x] 1.1 Add `onFsPacketReceived`, `onOtaPacketReceived`, and `onSettingsPacketReceived` assignments to the early-return path in `DeviceProvider.setTransport()` (radiokit-app/lib/providers/device_provider.dart, lines ~703-708)

## 2. Verify

- [x] 2.1 Run `flutter analyze --fatal-warnings` to check for static analysis errors
- [x] 2.2 Run `flutter test` to ensure no regressions
