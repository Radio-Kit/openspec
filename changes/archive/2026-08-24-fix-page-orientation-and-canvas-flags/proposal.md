## Why

When connecting to a multi-page device where the initial active page is not page 0 (or has a per-page orientation override), the Flutter companion app currently defaults to the global orientation (Landscape) and fails to apply the active page's orientation until a tab is clicked manually. Additionally, canvas display preferences (such as hiding the page tab switcher in play/control mode) are not transmitted over the wire, causing the app to rely on local design name matching which fails if the device name differs from the local template name.

Transmitting a 1-byte canvas flags field in the `CONF_DATA` binary payload and ensuring `DeviceProvider` resolves `_orientation` from `_pageOrientations[_activePage]` on connection ensures immediate, zero-latency synchronization and accurate UI rendering regardless of device name.

## What Changes

- **Connection Orientation Synchronization**: On `CONF_DATA` receipt, `DeviceProvider` immediately applies `_orientation = _pageOrientations[_activePage]` and generates the cached designer JSON using the active page orientation.
- **Wire Protocol Canvas Flags**: Add a 1-byte `canvasFlags` field to `CONF_DATA` (bit 0 = `showPageBar`, bit 1 = `showControlPageBar`, default = `0x03`).
- **Firmware API & State**: Add `RadioKit.setCanvasFlags(uint8_t flags)` to `RadioKitClass` in `rk-arduino` and serialize `_canvasFlags` in `CONF_DATA`.
- **Arduino Code Generation**: Emit `RadioKit.setCanvasFlags(...)` during `RADIOKIT.h` codegen when `showPageBar` or `showControlPageBar` differs from default.
- **App Protocol Parsing & UI Gating**: Update `ProtocolService.parseConfData` to read `canvasFlags`, pass them into `widgetConfigsToDesignerJson`, and gate `PageSwitcher` rendering accordingly.

## Capabilities

### New Capabilities
<!-- None -->

### Modified Capabilities
- `page-orientation-override`: Update connection behavior requirement so that the active page's effective orientation is applied immediately upon receiving initial `CONF_DATA`.
- `multi-page-protocol`: Add `canvasFlags` (1 byte bitmask) to the `CONF_DATA` payload structure.
- `page-bar-visibility`: Update requirement to support wire-transmitted canvas flags in addition to JSON persistence for controlling `showPageBar` and `showControlPageBar`.

## Impact

- **Firmware (`rk-arduino`)**: `RadioKitClass.h` gains `setCanvasFlags()`; `RadioKit.cpp` appends `_canvasFlags` to `CONF_DATA` frame.
- **App (`radiokit-app`)**:
  - `lib/services/protocol_service.dart`: `parseConfData` and `DeviceConfig` updated to parse `canvasFlags`.
  - `lib/providers/device_provider.dart`: `_handleConfData` orientation resolution fixed; `canvasFlags` passed into designer JSON.
  - `lib/screens/designer/codegen/json_arduino_generator.dart`: `setCanvasFlags()` generated in `setup()`.
- **Backward Compatibility**: `ProtocolService.parseConfData` treats missing `canvasFlags` as `0x03` (both page bars visible by default).
