## 1. Library Optimization (`rk-arduino`)

- [x] 1.1 Remove automatic `rk_buildAck` / `_sendPacket` generation from `RadioKitClass::_handleVarUpdate` in `rk-arduino/src/RadioKit.cpp`.

## 2. App BLE Subscription & Dispatch Optimization (`radiokit-app`)

- [x] 2.1 Update `_connectDevice` in `lib/services/ble_service_impl.dart` to subscribe to discovered characteristics sequentially with `await` instead of `Future.wait`.
- [x] 2.2 Update multi-device characteristic subscription in `lib/services/ble_service_impl.dart` to subscribe sequentially with `await`.
- [x] 2.3 Verify flutter tests and analyze files to ensure clean compilation.
