## 1. Arduino Library & Firmware

- [x] 1.1 Add `_canvasFlags` and `setCanvasFlags(uint8_t flags)` to `RadioKitClass.h` along with `RK_CANVAS_SHOW_PAGE_BAR (0x01)`, `RK_CANVAS_SHOW_CONTROL_PAGE_BAR (0x02)`, and `RK_CANVAS_DEFAULT_FLAGS (0x03)`.
- [x] 1.2 Update `RadioKit.cpp` `CONF_DATA` builder to serialize `_canvasFlags` byte after the `_pageOrientations` array.

## 2. Arduino Codegen

- [x] 2.1 Update `json_arduino_generator.dart` to compute canvas flags from `showPageBar` and `showControlPageBar` and emit `RadioKit.setCanvasFlags(...)` during `setup()`.
- [x] 2.2 Add unit tests in `json_arduino_generator_test.dart` verifying `setCanvasFlags` emission for custom canvas configurations.

## 3. Protocol & App State

- [x] 3.1 Update `DeviceConfig` in `protocol_service.dart` to include `canvasFlags` (default `0x03`).
- [x] 3.2 Update `ProtocolService.parseConfData` to parse `canvasFlags` when present after `pageOrientations`.
- [x] 3.3 Fix `DeviceProvider._handleConfData` to immediately resolve `_orientation = _pageOrientations[_activePage]` on connection.
- [x] 3.4 Update `DeviceProvider._handleConfData` to apply `conf.canvasFlags` directly to `showPageBar` and `showControlPageBar` in `widgetConfigsToDesignerJson`.
- [x] 3.5 Pass the active page's effective `_orientation` into `widgetConfigsToDesignerJson` during `CONF_DATA` handling.

## 4. Testing & Verification

- [x] 4.1 Add unit tests for `ProtocolService.parseConfData` verifying `canvasFlags` parsing and backward compatibility.
- [x] 4.2 Add unit tests for `DeviceProvider` connection orientation synchronization.
- [x] 4.3 Run `flutter test` across `radiokit-app` to verify all test suites pass.
