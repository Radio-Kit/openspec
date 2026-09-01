## Why

Two issues currently degrade real-time performance and connection reliability during BLE operations:
1. **BLE ACK Flood on Continuous Inputs**: On every touch move / slider drag (e.g. 40 updates/sec), `RadioKitClass::_handleVarUpdate` builds an ACK packet and transmits a BLE notification back to the client. This saturates the BLE notification buffer, causes queue contention with telemetry frames, and induces MCU loop jitter.
2. **GATT Error 17 on BLE Connect**: The Android Bluetooth stack cannot process multiple concurrent descriptor write operations. Subscribing to 5 characteristics in parallel with `Future.wait` causes `GATT_BUSY` (status 17) errors during connection setup.

## What Changes

- **Library (`rk-arduino`)**: Remove automatic ACK notification generation from `RadioKitClass::_handleVarUpdate`. Real-time continuous inputs rely on state synchronization and write-without-response rather than individual packet ACKs.
- **App (`radiokit-app`)**: Change characteristic notification subscriptions from parallel `Future.wait` to sequential `await` loops in both single-device and multi-device connection handlers in `ble_service_impl.dart`.
- **App (`radiokit-app`)**: Streamline `_sendVarUpdate` in `device_provider.dart` for streaming inputs.

## Capabilities

### New Capabilities
- `ble-var-update-no-ack`: Suppress redundant BLE ACK notifications on variable updates to eliminate reverse packet floods and prevent GATT TX queue saturation.

### Modified Capabilities
- `parallel-ble-subscriptions`: Change characteristic notification subscriptions from parallel `Future.wait` to sequential `await` loops to comply with Android single-operation GATT descriptor constraints.

## Impact

- `rk-arduino`: `src/RadioKit.cpp` (`_handleVarUpdate`) eliminates ACK packet creation/sending.
- `radiokit-app`: `lib/services/ble_service_impl.dart` (`_connectDevice`, `_subscribeCharacteristicsMulti`) uses sequential subscription.
- `radiokit-app`: `lib/providers/device_provider.dart` (`_sendVarUpdate`).
- Latency & Reliability: GATT 17 errors during connection are eliminated, and BLE notification queue congestion during touch drags is eliminated.
