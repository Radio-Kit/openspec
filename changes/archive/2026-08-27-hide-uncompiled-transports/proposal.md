## Why

When firmware is compiled without certain transports enabled (e.g. `RK_ENABLE_WIFI` or `RK_ENABLE_BLE` not defined), the companion app's Settings Tab still displays those transports as disabled toggles, giving the false impression that users can enable them at runtime. Furthermore, write operations to NVS for uncompiled transports should be rejected with safety guards. Gating transport settings by compile-time feature flags creates a clean, accurate UX and prevents invalid state modifications.

## What Changes

- **Firmware Feature Flag Reporting**: In `RadioKitClass::_handleSettingsGetFeatures()`, report `RK_SETTINGS_FEATURE_WIFI` and `RK_SETTINGS_FEATURE_CLOUD` based on compile-time macros (`#if defined(RK_ENABLE_WIFI)` / `#if defined(RK_ENABLE_CLOUD)`), consistent with `RK_SETTINGS_FEATURE_BLE`.
- **Firmware NVS Write Guard**: Guard NVS write operations for transport enable keys (`rk_wifi_on`, `rk_ble_on`, `rk_cloud_on`) so writes for uncompiled transports return an error status.
- **Companion App Settings UI Gating**: In `SettingsTab` (`settings_tab.dart`), conditionally render transport sections (BLE, WiFi, Cloud) only if the connected device's capability bitmask indicates the feature is compiled (`dp.hasBle`, `dp.hasWifi`, `dp.hasCloud`).
- **NVS Read Optimization**: Skip querying NVS keys (`rk_wifi_on`, `rk_sta_ssid`, `rk_sta_pwd`, `rk_cloud_url`, `rk_cloud_account`) when `!dp.hasWifi` to avoid unnecessary BLE round-trips upon opening the settings bottom sheet.

## Capabilities

### New Capabilities
- `transport-settings-gating`: Gates transport configuration options in the app UI and enforces firmware safety guards based on compile-time transport capabilities.

### Modified Capabilities
<!-- None -->

## Impact

- `RadioKit` firmware: `_handleSettingsGetFeatures()` and `_handleSettingsNvsRawWrite()` in `RadioKit.cpp`.
- `radiokit-app`: `SettingsTab` in `lib/screens/device_config/settings_tab.dart`.
- Protocols: Settings Protocol (`0xDD`) feature bitmask semantics.
