## Context

`DeviceProvider.setTransport()` has two code paths:

1. **Full path** (new transport instance): wraps in `DebugTransport` if needed, then sets all five callbacks.
2. **Early-return path** (same transport instance): only updates `onPacketReceived` and `onConnectionLost`, returning before setting the other three.

The constructor always hits the early-return path because `_transport` is assigned the same instance in the initializer list before `setTransport()` is called in the constructor body. This means `onSettingsPacketReceived`, `onFsPacketReceived`, and `onOtaPacketReceived` are never wired up on first connection.

The bug is silent because:
- Widget packets (CONF_DATA, VAR_UPDATE) flow through `onPacketReceived` — works fine.
- Settings packets (FEATURES_DATA, TELEMETRY_DATA, BLE_INFO_DATA) flow through `onSettingsPacketReceived` — dropped.
- FS packets flow through `onFsPacketReceived` — dropped.
- OTA packets flow through `onOtaPacketReceived` — dropped.

## Goals / Non-Goals

**Goals:**
- Ensure all five transport callbacks are set on every `setTransport()` call, regardless of which path is taken.
- Minimal diff — one method, no new abstractions.

**Non-Goals:**
- Refactoring the transport setup to avoid the double-assignment in the constructor.
- Changing the public API of `DeviceProvider` or `TransportService`.

## Decisions

**Decision: Add the three missing callback assignments to the early-return path.**

The early-return path should set the same five callbacks as the full path. No conditional logic needed — unconditionally assigning all callbacks is safe (idempotent) and eliminates the class of bugs where a new callback is added to the full path but forgotten in the early return.

```dart
// Before (buggy):
if (identical(currentBase, base) && hasCorrectLayers) {
  _transport.onPacketReceived = _handlePacket;
  _transport.onConnectionLost = _handleConnectionLost;
  return;
}

// After (fixed):
if (identical(currentBase, base) && hasCorrectLayers) {
  _transport.onPacketReceived = _handlePacket;
  _transport.onFsPacketReceived = _handleFsPacket;
  _transport.onOtaPacketReceived = _handleOtaPacket;
  _transport.onSettingsPacketReceived = _handleSettingsPacket;
  _transport.onConnectionLost = _handleConnectionLost;
  return;
}
```

**Alternative considered: Remove the early-return path entirely.**
Rejected because the early-return exists to avoid redundant `DebugTransport` wrapping. Removing it would wrap the same transport in duplicate `DebugTransport` layers on every `setTransport()` call after the first.

## Risks / Trade-offs

- **Risk**: None significant. The fix is a pure addition of three line assignments. If any of these callbacks were already set by some other code path, re-assigning the same handler is harmless.
- **Trade-off**: Slightly more duplication between the two paths. A future refactor could extract a `_applyCallbacks()` helper to keep them in sync, but that's out of scope for this fix.
