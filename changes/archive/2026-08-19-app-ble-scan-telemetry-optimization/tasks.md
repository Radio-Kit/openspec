## 1. BLE Scan Lifecycle Management

- [x] 1.1 Guard `startScan()` in `lib/services/ble_service_impl.dart` to return an empty stream and skip scanning when a device is already connected (`_connectedDeviceId != null`).
- [x] 1.2 Ensure `BleProvider.stopScan()` is called and cancels `_scanLoopTimer` when connecting.

## 2. Telemetry Output Sizing & Inbound Processing

- [x] 2.1 Add `kWidgetTelemetry: 33` to `kWidgetOutputSize` in `lib/models/protocol.dart`.
- [x] 2.2 Update `_handleVarUpdate` in `lib/providers/device_provider.dart` to decode `kWidgetTelemetry` and store the result in `_telemetryValues`.
- [x] 2.3 Remove reverse `_writePacket(ProtocolService.buildAck(seq))` calls from `_handleVarUpdate`, `_handleSetInput`, and `_handleMetaUpdate` in `lib/providers/device_provider.dart`.
- [x] 2.4 Run flutter test suite and verify 0 errors.
