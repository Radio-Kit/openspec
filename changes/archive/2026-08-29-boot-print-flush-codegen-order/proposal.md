## Why

Three issues were discovered during BLE connection debugging on RC_brain (TRACKLINK_V3 with ESP32-S3):

1. **Boot messages invisible**: `RadioKit.print()` buffers messages but `_flushPrintBuffer()` gates behind `isConnected()`, making all BLE init output invisible on serial during boot. This made debugging the NimBLE crash nearly impossible.

2. **initRadioKit() init order**: The codegen generates `startBLE()` before `enableFS()`, causing LittleFS to mount after BLE is already running. On ESP32-S3 with XMC flash, this double-mount corrupts flash DMA. A safety guard was added to the library (fix #2), but the root cause is the init order.

3. **Locomotive test assertions**: Integration tests assume truck-specific features (horn_button, features.ota/filesystem keys), failing when run against locomotive configs.

## What Changes

- **Boot print flush**: Add a one-shot raw `Serial.write()` path in `_flushPrintBuffer()` that outputs buffered boot messages before the first client connects, then switches to the normal gated behavior.

- **Codegen init order**: Move `enableFS()` before `startBLE()` in generated `initRadioKit()` so the filesystem mounts before BLE starts, eliminating the double-mount race.

- **Locomotive test handling**: Update integration tests to detect vehicle type from config and adjust expectations (bell vs horn, feature keys).

## Capabilities

### New Capabilities
- `boot-print-diagnostics`: Boot-time RadioKit.print() messages are visible on raw serial output for debugging.
- `codegen-safe-init-order`: Generated initRadioKit() mounts filesystem before starting BLE, preventing flash DMA corruption on ESP32-S3.

### Modified Capabilities
- `locomotive-integration-tests`: Tests now handle both truck and locomotive vehicle types.

## Impact

- `rk-arduino/src/RadioKit.cpp`: _flushPrintBuffer() boot path
- `radiokit-app/lib/screens/designer/codegen/json_arduino_generator.dart`: initRadioKit() order
- `radiokit-app/test/`: test assertions for locomotive configs
