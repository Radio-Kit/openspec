## Why

Currently, firmware OTA updates rely on the user-editable `config.name` to match release binary assets on GitHub. When a user renames their device, asset matching fails or falls back to arbitrary binaries. Furthermore, releases often bundle full flash images alongside OTA-specific binaries (`*-ota.bin`), causing potential confusion when choosing assets to flash over BLE/WiFi.

Defining dedicated compile-time macros (`RK_BOARD` and `RK_VERSION`) guarantees that board identity and firmware version are immutable properties of the compiled binary, while prioritizing `-ota` binary assets in the companion app ensures safe and seamless updates.

## What Changes

- **Compile-Time Board & Version Macros**: Introduce `#define RK_BOARD` (default `"ESP32_GENERIC"`) alongside `#define RK_VERSION` in `RadioKitConfig.h`.
- **Hardware Target in `RK_Config`**: Add `const char* board = RK_BOARD;` in `RK_Config`.
- **BREAKING Settings Protocol Frame (0x88)**: Update `SETTINGS_RESP_DEVICE_INFO_DATA` (0x88) to send both `board` and `version` as length-prefixed strings.
- **Companion App Board & Version Decoders**: Parse `board` in `SettingsProtocolService`, expose it via `DeviceProvider`, and display it in the Firmware tab.
- **Smart `-ota` Binary Filtering**: Filter GitHub release assets to only show `-ota` binaries (`*-ota.bin` or `*_ota.bin`) by default if present, and match them against the device's board identifier.
- **"Show All" Asset Toggle**: Provide an inline toggle in the Firmware tab allowing users to view and select all attached release binaries.

## Capabilities

### Modified Capabilities
- `firmware-update-check`: Device info frame reports `board` and `firmwareVersion`; companion app matches release assets against `board`, filters `-ota` binaries by default, and provides a toggle to display all binaries.

## Impact

- `rk-arduino/src/RadioKitConfig.h`: Adds `RK_BOARD`, `RADIOKIT_MAX_BOARD`.
- `rk-arduino/src/RadioKitClass.h`: Adds `board` field to `RK_Config`.
- `rk-arduino/src/RadioKit.cpp`: Serializes `board` in `_handleSettingsDeviceInfo()`.
- `radiokit-app/lib/services/settings_protocol_service.dart`: Deserializes `board` in `parseDeviceInfoData()`.
- `radiokit-app/lib/providers/device_provider.dart`: Exposes `board`.
- `radiokit-app/lib/services/firmware_release_service.dart`: Implements `-ota` filtering and board matching.
- `radiokit-app/lib/screens/device_config/firmware_tab.dart`: Updates UI to show board name and "Show All" toggle.
