## Why

When connecting to an ESP32 device via Bluetooth LE, users experience an avoidable delay of 1.5 to 4 seconds before the CONTROLLER button and telemetry widgets become active. This delay is caused by two bottlenecks:
1. Characteristic notification subscriptions are executed sequentially over 5 individual round-trips (`Widget`, `FS`, `OTA`, `Settings`, `Print`), adding 500ms–800ms of serialized GATT descriptor writes.
2. If the initial proactive config push is missed or delayed, the handshake timeout (`kPushWaitTimeout`) forces the app into a 3-second passive wait before falling back to sending `GET_CONF`.

Optimizing both bottlenecks reduces connection time to under ~350ms–750ms total.

## What Changes

- **Parallelized Characteristic Subscriptions**: Update `BleService` (`connect` and `connectToDevice`) to subscribe to all discovered characteristics concurrently using `Future.wait`.
- **Fast Handshake Fallback**: Reduce `kPushWaitTimeout` from 3.0s down to 500ms in `protocol.dart` so that any delayed or missed push triggers an immediate `GET_CONF` query.
- **Immediate GET_CONF / Idempotent Handshake Protection**: Ensure `DeviceProvider` handles early or duplicate `CONF_DATA` packets smoothly and cleanly.

## Capabilities

### New Capabilities
- `parallel-ble-subscriptions`: Subscribes to all discovered RadioKit BLE characteristics concurrently on connection rather than awaiting each sequentially.

### Modified Capabilities
- `config-push-on-connect`: Shorten the BLE push wait window from 3 seconds to 500ms for fast fallback to `GET_CONF`.

## Impact

- `lib/services/ble_service_impl.dart`: Parallelize characteristic subscriptions in `connect()` and `connectToDevice()`.
- `lib/models/protocol.dart`: Update `kPushWaitTimeout` constant to 500ms.
- `lib/providers/device_provider.dart`: Ensure smooth fallback and verify logging.
