## Why

Currently, updating device firmware via OTA requires the user to manually compile, export, and select a local `.bin` file on their filesystem. Devices already configure an OTA repository link (`RK_OTA_URL` / `config.ota_url`), but the companion app cannot automatically check whether newer firmware releases exist. Enabling the Firmware Tab to query GitHub releases, compare the running firmware version, display changelogs, and download & flash matching binary assets provides a seamless, manual one-tap update experience.

## What Changes

- **Firmware Version in Protocol**: Append firmware version (`RadioKit.config.version`) to `SETTINGS_RESP_DEVICE_INFO_DATA` (0x88) in firmware and expose `dp.firmwareVersion` in `DeviceProvider`.
- **GitHub Release Service**: Add `FirmwareReleaseService` in the companion app to parse GitHub OTA URLs, fetch release metadata from GitHub Releases API (`/releases/latest`), extract version tags, parse changelogs, and download binary assets directly into memory.
- **Smart Asset Matching**: Auto-match release assets (`.bin` files) against the connected device's name or model with a fallback dropdown selector when multiple target board binaries exist.
- **Firmware Tab UI Overhaul**: Update `FirmwareTabContent` to display current firmware version, auto-check for updates if an OTA URL is configured (with manual refresh button), render release cards with expandable changelogs, and provide a "Download & Flash" button that streams the binary directly into the OTA transfer pipeline.
- **API Server & End-to-End Testing**: Provide HTTP API endpoint(s) if needed and perform end-to-end testing against real hardware (Mikro board) using test firmware hosted in GitHub releases (`https://github.com/Radio-Kit/demo-fs-assets`).

## Capabilities

### New Capabilities
- `firmware-update-check`: Automated checking of remote GitHub releases for firmware updates, version comparison, asset selection, and in-app binary retrieval for OTA flashing.

### Modified Capabilities
<!-- None -->

## Impact

- `RadioKit` firmware: `_handleSettingsDeviceInfo()` in `RadioKit.cpp` and `RadioKitSettings.h`.
- `radiokit-app`:
  - `lib/services/firmware_release_service.dart` (new)
  - `lib/providers/device_provider.dart` (`firmwareVersion` parsing and caching)
  - `lib/screens/device_config/firmware_tab.dart` (UI overhaul for update checks, release card, and direct download-and-flash)
  - Test suite / test harness for remote API and hardware verification.
