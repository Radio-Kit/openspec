## 1. Firmware Updates

- [x] 1.1 Update `RadioKitClass::_handleSettingsGetFeatures()` in `RadioKit.cpp` to report `RK_SETTINGS_FEATURE_WIFI` and `RK_SETTINGS_FEATURE_CLOUD` based on compile-time defines (`#if defined(RK_ENABLE_WIFI)` and `#if defined(RK_ENABLE_CLOUD)`).
- [x] 1.2 Add safety guards to `RadioKitClass::_handleSettingsNvsRawWrite()` in `RadioKit.cpp` rejecting writes to uncompiled transport keys (`rk_wifi_on`, `rk_sta_ssid`, `rk_sta_pwd`, `rk_ble_on`, `rk_cloud_url`, `rk_cloud_account`).

## 2. Companion App UI & Service Updates

- [x] 2.1 Gate transport rows and cards in `SettingsTab` (`settings_tab.dart`) using `dp.hasBle`, `dp.hasWifi`, and `dp.hasCloud`.
- [x] 2.2 Optimize `_loadTransportNvsKeys()` in `settings_tab.dart` to skip NVS queries for uncompiled transports.

## 3. Verification & Testing

- [x] 3.1 Verify with MIKRO_V2 build (non-WiFi): confirm WiFi card is completely hidden in SettingsTab and NVS write guards reject invalid writes.
- [x] 3.2 Sync updated library to RC_brain.
