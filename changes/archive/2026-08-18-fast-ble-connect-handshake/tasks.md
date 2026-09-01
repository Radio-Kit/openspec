## 1. Parallelize BLE Characteristic Subscriptions

- [x] 1.1 In `lib/services/ble_service_impl.dart`, update `connect()` to subscribe to all discovered characteristics (`_charWidgetId`, `_charFsId`, `_charOtaId`, `_charSettingsId`, `_charPrintId`) concurrently via `Future.wait`.
- [x] 1.2 In `lib/services/ble_service_impl.dart`, update `connectToDevice()` to subscribe to all discovered characteristics concurrently via `Future.wait`.

## 2. Fast Push Timeout Configuration

- [x] 2.1 In `lib/models/protocol.dart`, change `kPushWaitTimeout` from `Duration(seconds: 3)` to `Duration(milliseconds: 500)`.
- [x] 2.2 In `lib/providers/device_provider.dart`, verify push wait logging and fallback path to `GET_CONF`.

## 3. Verification & Validation

- [x] 3.1 Run Flutter analyzer/tests to ensure zero regressions in protocol parsing and BLE service.
- [x] 3.2 Verify fast handshake behavior and state transitions.
