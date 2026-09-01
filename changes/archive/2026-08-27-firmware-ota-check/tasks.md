## 1. Firmware Protocol Updates

- [x] 1.1 Update `RadioKitClass::_handleSettingsDeviceInfo()` in `RadioKit.cpp` to append `RadioKit.config.version` to `SETTINGS_RESP_DEVICE_INFO_DATA` (0x88).
- [x] 1.2 Support custom version macro in sketches/PlatformIO (e.g. `RK_VERSION` / `RadioKit.config.version`).

## 2. Companion App Service & Protocol

- [x] 2.1 Update `SettingsProtocolService.parseDeviceInfoData` and `DeviceProvider` in `radiokit-app` to parse and expose `dp.firmwareVersion`.
- [x] 2.2 Implement `FirmwareReleaseService` in `radiokit-app/lib/services/firmware_release_service.dart` to query GitHub Releases, parse semver tags/changelogs, match board `.bin` assets, and stream binary downloads.
- [x] 2.3 Add unit tests for `FirmwareReleaseService` in `radiokit-app/test/services/firmware_release_service_test.dart`.

## 3. Companion App UI (Firmware Tab)

- [x] 3.1 Update `FirmwareTabContent` in `radiokit-app/lib/screens/device_config/firmware_tab.dart` to display current firmware version, auto-check updates on tab open, and provide a manual refresh action.
- [x] 3.2 Build the Update Available card in `FirmwareTabContent` with new version badge, release date, asset picker dropdown, and expandable changelog viewer.
- [x] 3.3 Wire "DOWNLOAD & FLASH" button to download binary in-memory and trigger the existing OTA flashing pipeline (`dp.uploadFirmware()`).

## 4. End-to-End Testing & Hardware Verification

- [x] 4.1 Prepare test firmware binary with version tag in `https://github.com/Radio-Kit/demo-fs-assets` GitHub releases.
- [x] 4.2 Perform end-to-end test on the connected Mikro board: verify update detection, asset download, OTA flash, reboot, and version verification.
