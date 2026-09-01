## Why

Three issues in the Flutter companion app contribute to significant latency and packet congestion during active BLE operation:
1. **Unstopped Background Scanning**: `BleProvider` runs a perpetual 4s scan / 4s idle timer loop. Connecting to a device did not cancel `BleProvider`'s timer, forcing Android to continuously scan advertising channels in the background and starving the active GATT connection.
2. **Reverse ACK Flooding**: `device_provider.dart` automatically sends `buildAck` packets back to the device on every incoming `VAR_UPDATE`. When the MCU sends telemetry every 100ms, the app replies with ACKs over BLE, creating continuous bidirectional congestion.
3. **Telemetry Widget Classification**: `kWidgetTelemetry` (`0x0A`) was missing from `kWidgetOutputSize` in `protocol.dart`, causing battery and speed telemetry to have `outputSize = 0` and be discarded as "IGNORED BOUNCE for Input".

## What Changes

- **Halt Background Scanning on Connect**: Cancel `BleProvider`'s scan loop timer on connection and guard `BleService.startScan()` against running while connected.
- **Suppress Reverse ACKs on Incoming Updates**: Remove automatic `buildAck` packet emissions from `_handleVarUpdate`, `_handleSetInput`, and `_handleMetaUpdate` in `device_provider.dart`.
- **Register Telemetry Output Size**: Add `kWidgetTelemetry: 33` to `kWidgetOutputSize` in `protocol.dart` and decode telemetry text in `_handleVarUpdate`.

## Capabilities

### New Capabilities
- `ble-scan-lifecycle`: Ensure background BLE scanning is strictly stopped and prevented while a device connection is active.
- `app-telemetry-output-sync`: Correct output sizing for `kWidgetTelemetry` and suppress redundant app-to-device ACKs on incoming state updates.

### Modified Capabilities
<!-- None -->

## Impact

- `radiokit-app`: `lib/providers/ble_provider.dart`, `lib/services/ble_service_impl.dart`, `lib/providers/device_provider.dart`, `lib/models/protocol.dart`.
- BLE Link Performance: Radio channel hopping eliminated during drive sessions, bidirectional frame rate halved, and telemetry readouts function cleanly.
