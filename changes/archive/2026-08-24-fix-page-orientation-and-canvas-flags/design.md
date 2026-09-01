## Context

In multi-page RadioKit projects, the active page on boot might have a portrait orientation override (e.g. Page 1 for a Locomotive vs Page 0 for a Truck). Currently, `DeviceProvider` sets `_orientation = conf.orientation` (global orientation 0 / Landscape) on connection and never updates `_orientation` to `_pageOrientations[_activePage]`. This keeps the UI locked in landscape mode until the user manually triggers a tab switch.

Furthermore, designer-level canvas flags like `showControlPageBar` are not sent over the wire in `CONF_DATA`. The app attempts to hydrate these flags by querying `_designSeedForConfig(configName)` to find a saved design in local storage matching the device name. When the device name (e.g., "TrackLink Loco") does not match the saved design name (e.g., "RC UI"), the seed lookup returns `null` and `PageSwitcher` defaults to showing the page tab bar.

## Goals / Non-Goals

**Goals:**
- Fix `DeviceProvider` connection handshake to immediately set `_orientation` to `_pageOrientations[_activePage]` and pass the effective orientation into the cached designer JSON reconstruction.
- Introduce a 1-byte `canvasFlags` bitmask into the `CONF_DATA` frame (Bit 0: `showPageBar`, Bit 1: `showControlPageBar`).
- Add `RadioKit.setCanvasFlags(uint8_t flags)` to `RadioKitClass` in `rk-arduino`.
- Update `JsonArduinoGenerator` in the Flutter app to emit `RadioKit.setCanvasFlags(...)` when `showPageBar` or `showControlPageBar` differs from default.
- Update `ProtocolService.parseConfData` and `DeviceProvider` to unpack `canvasFlags` and use them for UI gating without depending on local template name matching.

**Non-Goals:**
- Transmitting full designer JSON configs over BLE or implementing a LittleFS JSON parser in runtime firmware.
- Changing widget layout coordinate systems or multi-page switching state machines.

## Decisions

### Decision 1: Resolve active page orientation immediately in `_handleConfData`
- **Choice**: In `DeviceProvider._handleConfData`, after resolving `_pageOrientations` and `_activePage`, execute `if (_activePage < _pageOrientations.length) _orientation = _pageOrientations[_activePage];` and ensure `widgetConfigsToDesignerJson` receives this effective orientation.
- **Rationale**: Ensures the screen orientation matches the MCU's active page from the very first frame rendered upon connection.

### Decision 2: 1-Byte `canvasFlags` field in `CONF_DATA`
- **Choice**: Append a single `uint8_t canvasFlags` after the `pageOrientations` array in `CONF_DATA`.
  - Bit 0 (`0x01`): `RK_CANVAS_SHOW_PAGE_BAR`
  - Bit 1 (`0x02`): `RK_CANVAS_SHOW_CONTROL_PAGE_BAR`
  - Default: `0x03` (both enabled)
- **Rationale**: Minimal wire overhead (+1 byte), zero MCU JSON parsing requirements, works seamlessly over BLE.
- **Alternatives Considered**:
  - *Storing and reading full JSON from LittleFS*: Overkill for canvas display toggles, adds BLE transfer latency, requires flash partitioning.
  - *App-side design alias mapping*: Requires manual user configuration in UI and breaks if templates are not imported.

### Decision 3: Backward compatible parsing in `ProtocolService`
- **Choice**: In `ProtocolService.parseConfData`, check if an extra byte exists after `pageOrientations`. If present, read `canvasFlags`; if absent (older firmware), default to `0x03`.
- **Rationale**: Prevents crashes or regressions when connecting to older firmware builds.

## Risks / Trade-offs

- **[Risk]** Older firmware sending `CONF_DATA` without `canvasFlags`.
  → **Mitigation**: `ProtocolService.parseConfData` performs offset bounds checking and defaults `canvasFlags = 0x03`.
- **[Risk]** Single-page configs vs multi-page configs.
  → **Mitigation**: When `numPages <= 1`, `pageOrientations` is empty and `PageSwitcher` is hidden regardless of flags.
