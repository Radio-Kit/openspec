## Context

Android BLE performance is heavily impacted by concurrent radio scanning during active GATT connections. In `radiokit-app`, `BleProvider` ran a periodic 4-second scan loop that was never stopped on connection. Furthermore, incoming telemetry frames (`VAR_UPDATE` for `kWidgetTelemetry`) were misinterpreted as input bounces due to a missing mapping in `kWidgetOutputSize`, and triggered automatic reverse `buildAck` packet emissions from `DeviceProvider` every 100ms.

## Goals / Non-Goals

**Goals:**
- Cancel `BleProvider`'s recurring scan loop upon connecting to a device, and guard `BleService.startScan()` against scanning while connected.
- Add `kWidgetTelemetry: 33` to `kWidgetOutputSize` in `protocol.dart`.
- Update `_handleVarUpdate` in `device_provider.dart` to decode `kWidgetTelemetry` and store its string value into `_telemetryValues`.
- Remove reverse `_writePacket(buildAck)` from `_handleVarUpdate`, `_handleSetInput`, and `_handleMetaUpdate` in `device_provider.dart`.

**Non-Goals:**
- Modifying the underlying `UniversalBle` platform channels.
- Changing `DeviceFsService` or OTA flow which use their own sub-protocols.

## Decisions

### Decision 1: Strict Scan Guards
- In `BleService.startScan()`, return an empty stream if `_connectedDeviceId != null`.
- In `BleProvider`, expose a clear `stopScan()` method and ensure `DeviceProvider` / connection callbacks trigger `stopScan()`.

### Decision 2: Wire Format Sizing for Telemetry
- In `protocol.dart`, `kWidgetOutputSize` maps `kWidgetTelemetry: 33` (matching firmware `RADIOKIT_TEXT_LEN + 1`).
- In `device_provider.dart`, `_handleVarUpdate` checks `if (widget.typeId == kWidgetText || widget.typeId == kWidgetTelemetry)` to decode and update `_telemetryValues`.

### Decision 3: Zero-ACK Inbound Processing
- Remove `_writePacket(ProtocolService.buildAck(seq))` from `_handleVarUpdate`, `_handleSetInput`, and `_handleMetaUpdate`.

## Risks / Trade-offs

- None identified; eliminating background scan loops and reverse ACKs significantly frees BLE radio bandwidth.
