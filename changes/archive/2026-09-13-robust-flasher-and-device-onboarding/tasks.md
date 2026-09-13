## 1. Flasher Provider & Protocol

- [x] 1.1 Implement single-pass unified compressed binary streaming in `FlasherProvider.startFlashing()`
- [x] 1.2 Implement 1200-baud Native USB CDC reset touch in `_enterBootloaderMode()`
- [x] 1.3 Add timeout safety and actionable manual bootloader tip to `connect()`

## 2. BLE Discovery & Permissions

- [x] 2.1 Update `BleService.startScan()` in `ble_service_impl.dart` to discover devices with or without `RK_` prefix
- [x] 2.2 Verify Android Bluetooth and Location runtime permissions request flow

## 3. Arduino Example Sketch Hardening

- [x] 3.1 Remove blocking `while (!Serial)` from `BasicSwitch.cpp` and all example sketches
- [x] 3.2 Ensure `LittleFS.begin(true)` self-formats cleanly on unformatted partitions

## 4. Verification & Testing

- [x] 4.1 Run unit tests across `radiokit-app` (`flutter test`)
- [x] 4.2 Build and verify `BasicSwitch` firmware binary bundle with PlatformIO
- [x] 4.3 Verify single-click connection and single-pass flashing via Remote API on connected board
