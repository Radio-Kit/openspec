## 1. Arduino C++ Library

- [x] 1.1 Add `RADIOKIT_MAX_BOARD` (32) and `#define RK_BOARD "ESP32_GENERIC"` macro to `rk-arduino/src/RadioKitConfig.h`
- [x] 1.2 Add `const char* board = RK_BOARD;` to `RK_Config` in `rk-arduino/src/RadioKitClass.h`
- [x] 1.3 Update `RadioKitClass::_handleSettingsDeviceInfo()` in `rk-arduino/src/RadioKit.cpp` to encode `board` before `version` in payload frame 0x88

## 2. Flutter Protocol & Services

- [x] 2.1 Update `SettingsProtocolService.parseDeviceInfoData()` to decode `board` and `firmwareVersion`
- [x] 2.2 Expose `board` getter and state in `DeviceProvider`
- [x] 2.3 Implement `-ota` asset filtering and "show all" logic in `FirmwareRelease.getFilteredBinAssets()` and `FirmwareRelease.findBestAsset()` in `FirmwareReleaseService`

## 3. Flutter UI & Integration

- [x] 3.1 Update `FirmwareTab` to display `BOARD` in the device info section
- [x] 3.2 Add "SHOW ALL" / "SHOW ONLY OTA" toggle button to the asset selection UI in `FirmwareTab`
- [x] 3.3 Auto-select the matching `-ota.bin` based on `dp.board`

## 4. Validation & Docs

- [x] 4.1 Update `SKILLS/radiokit-ota/SKILL.md` and related docs with `RK_BOARD` usage and `-ota` asset conventions
- [x] 4.2 Verify Flutter tests and compile check
