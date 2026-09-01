## Why

When firmware `RadioKitClass::_handleVarUpdate` was updated to suppress `RK_CMD_ACK` frames (spec `ble-var-update-no-ack`), `DeviceProvider._sendVarUpdate` in `radiokit-app` was left retaining an ACK-wait retry mechanism (5 retries × 200ms = 1,000ms). Because the firmware never replies with an ACK, every widget interaction entered a 1-second timeout loop before issuing a fallback `GET_VARS` query. In addition, Android BLE connection priority was left at the default 48ms interval, and `RKMultiButton` UI had a 300ms transition animation.

## What Changes

- **Direct Write-Without-Response in `_sendVarUpdate`**: Remove the blocking 5-retry ACK timeout in `DeviceProvider._sendVarUpdate`. Transmit `VAR_UPDATE` packets directly over `_writePacket(pkt)` for immediate delivery without waiting for non-existent ACKs.
- **High-Priority BLE Connection Mode**: Call `UniversalBle.requestConnectionPriority(deviceId, BleConnectionPriority.high)` upon BLE connection setup in `ble_service_impl.dart` to negotiate 11.25ms–15ms intervals on Android.
- **Snappy Multi-Button UI Transitions**: Reduce `AnimatedContainer` duration in `rk_multi_button.dart` from 300ms to 50ms for instant visual feedback.

## Capabilities

### Modified Capabilities
- `ble-var-update-no-ack`: Update client-side requirements to specify write-without-response transmission for `VAR_UPDATE` without ACK wait timers.
- `parallel-ble-subscriptions`: Add high connection priority requirement upon connection.

## Impact

- **App Files**: `lib/providers/device_provider.dart`, `lib/services/ble_service_impl.dart`, `../flutter-widgets/lib/src/widgets/multiple/rk_multi_button.dart`.
- **Latency**: Removes 1,000ms retry stall and cuts BLE radio latency by 3x.
