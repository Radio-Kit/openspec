## Context

RadioKit supports OTA firmware updates over BLE and WiFi via the companion Flutter app and GitHub releases. Previously, the app attempted to identify compatible firmware binary assets by comparing the release asset names against `dp.configName` (the user-editable device name). If the user changed their device name to "Desk Light", auto-selection failed. Furthermore, GitHub releases frequently bundle multiple binaries including full factory binaries alongside OTA update images.

## Goals / Non-Goals

**Goals:**
- Provide compile-time macros `RK_BOARD` and `RK_VERSION` to burn hardware target and firmware version into the binary.
- Transmit `board` and `firmware_version` as explicit, length-prefixed fields in the Settings protocol `SETTINGS_RESP_DEVICE_INFO_DATA` (0x88) response.
- Prioritize and filter `-ota` binary assets in the Flutter companion app's OTA update tab.
- Provide a "Show All" toggle in the Firmware tab allowing users to view and flash all binaries if needed.

**Non-Goals:**
- Backward compatibility with legacy pre-v5 settings protocol frame payloads.
- Automatic binary flashing without user confirmation.

## Decisions

### 1. Compile-Time Macros in `RadioKitConfig.h`
- **Decision**: Define `#define RK_BOARD "ESP32_GENERIC"` and `#define RK_VERSION "1.0.0"` in `RadioKitConfig.h`. Allow overriding via `-DRK_BOARD=...` in `platformio.ini` or in sketch headers before including RadioKit.
- **Rationale**: Hardware board models and firmware releases are intrinsic to the build environment and should not depend on runtime NVS state.

### 2. Explicit Protocol Payload Structure
- **Decision**: Send `[PROTO_VER][NAME_LEN][NAME][DESC_LEN][DESC][UID_LEN][UID][ICON_LEN][ICON][BOARD_LEN][BOARD][VER_LEN][VERSION]`.
- **Rationale**: Keeps the binary payload compact, clear, and unambiguous.

### 3. Smart Filter with Manual Fallback
- **Decision**: `FirmwareRelease.getFilteredBinAssets({bool showAll})` filters for assets containing `-ota` or `_ota` if present unless `showAll` is true. `FirmwareRelease.findBestAsset(boardName)` matches the board name within candidate assets.
- **Rationale**: Automates the 95% common case while giving developers and advanced users full manual control.

## Risks / Trade-offs

- **[Risk] Existing sketches without `RK_BOARD` defined**:
  - *Mitigation*: Defaults to `"ESP32_GENERIC"`. App falls back to matching on all `.bin` assets or manual dropdown selection if no exact board match is found.
- **[Risk] Release has no `-ota` naming convention**:
  - *Mitigation*: If no `-ota` binaries are found in the release assets, all `.bin` assets are automatically shown without filtering.
