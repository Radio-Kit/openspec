## Context

The firmware suppresses ACK frames for `VAR_UPDATE` packets to achieve maximum throughput and minimal overhead on BLE. However, `DeviceProvider` in `radiokit-app` still maintained a 5-retry × 200ms ACK retry timer that inevitably timed out after 1,000ms and sent a redundant `GET_VARS` query.

## Goals / Non-Goals

**Goals:**
- Make `_sendVarUpdate` direct and non-blocking in `DeviceProvider`.
- Negotiate high-priority BLE intervals (11.25ms–15ms) on Android.
- Reduce multi-button visual animation duration to 50ms.

**Non-Goals:**
- Changing bulk FS or OTA protocol behaviors (which continue to use ACKs).

## Decisions

- **Decision**: Update `_sendVarUpdate` to construct the packet and call `await _writePacket(pkt)` directly, discarding `_pendingUpdates` timer management.
- **Decision**: Invoke `UniversalBle.requestConnectionPriority(deviceId, BleConnectionPriority.high)` immediately after characteristic discovery in `ble_service_impl.dart`.
