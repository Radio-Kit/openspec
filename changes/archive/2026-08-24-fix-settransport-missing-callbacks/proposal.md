## Why

The `DeviceProvider.setTransport()` method has an early-return path that only wires up `onPacketReceived` and `onConnectionLost`, omitting `onSettingsPacketReceived`, `onFsPacketReceived`, and `onOtaPacketReceived`. Because the constructor passes the same transport instance to `setTransport()` as was assigned to `_transport`, the early return is always taken on first connection. This silently drops all settings protocol responses — including the features bitmask that advertises OTA, filesystem, password, and transport capabilities to the Flutter app.

## What Changes

- Fix the `setTransport()` early-return path to set all five callbacks (`onPacketReceived`, `onFsPacketReceived`, `onOtaPacketReceived`, `onSettingsPacketReceived`, `onConnectionLost`) when the transport instance is unchanged.

## Capabilities

### New Capabilities

(none — this is a bug fix)

### Modified Capabilities

(none — no spec-level behavior changes, only implementation correctness)

## Impact

- **Affected code**: `radiokit-app/lib/providers/device_provider.dart` — `setTransport()` method (lines ~694–708)
- **Symptoms fixed**: OTA tab not visible, features bitmask always zero, settings protocol responses silently dropped on all connections (BLE, WiFi, Serial, Cloud)
- **No API change**: public interface unchanged
- **No dependency changes**
