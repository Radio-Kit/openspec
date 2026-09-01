# Tasks: Fix VAR_UPDATE App Sync and BLE Latency

## 1. App Provider and Transport Updates

- [x] 1.1 Update `DeviceProvider._sendVarUpdate` in `radiokit-app/lib/providers/device_provider.dart` to send `VAR_UPDATE` directly via `_writePacket` without ACK retry timeouts
- [x] 1.2 Add `UniversalBle.requestConnectionPriority(deviceId, BleConnectionPriority.high)` in `radiokit-app/lib/services/ble_service_impl.dart` upon BLE connection discovery
- [x] 1.3 Reduce `AnimatedContainer` duration in `flutter-widgets/lib/src/widgets/multiple/rk_multi_button.dart` from 300ms to 50ms

## 2. Tests and Verification

- [x] 2.1 Run Flutter tests in `radiokit-app` (`flutter test`)
- [x] 2.2 Rebuild and install RadioKit app on Android tablet (`flutter build apk` / `adb install`)
