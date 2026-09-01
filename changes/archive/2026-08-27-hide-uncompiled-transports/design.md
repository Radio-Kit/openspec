## Context

RadioKit exposes runtime settings via the Settings Protocol (`0xDD`). Transport options such as BLE and WiFi/Cloud are configured in NVS keys (`rk_ble_on`, `rk_wifi_on`, `rk_cloud_on`).

Previously:
1. `RadioKit.cpp` tied the `RK_SETTINGS_FEATURE_WIFI` bit in `_handleSettingsGetFeatures()` to `_wifiActive` (runtime state), whereas `RK_SETTINGS_FEATURE_BLE` was tied to `#if defined(RK_ENABLE_BLE)`.
2. `settings_tab.dart` unconditionally displayed the WiFi card regardless of whether WiFi was compiled into firmware.
3. Firmware did not guard NVS raw writes for uncompiled transports.

## Goals / Non-Goals

**Goals:**
- Unify compile-time capability reporting in the features bitmask (`RK_SETTINGS_FEATURE_BLE`, `RK_SETTINGS_FEATURE_WIFI`, `RK_SETTINGS_FEATURE_CLOUD`).
- Hide uncompiled transport UI elements completely in `SettingsTab`.
- Prevent unnecessary NVS queries during sheet load for uncompiled transports.
- Add safety guards to `_handleSettingsNvsRawWrite()` rejecting writes to transport keys for uncompiled features.

**Non-Goals:**
- Changing the binary protocol framing of `0xDD` settings commands.
- Dynamic runtime loading/compilation of transports.

## Decisions

### Decision 1: Use Features Bitmask for UI Visibility
- **Rationale**: Feature flags are exchanged during the initial connection handshake and cached in `DeviceProvider`. Gating UI on `dp.hasBle`, `dp.hasWifi`, and `dp.hasCloud` provides instant rendering without waiting for NVS key lookups and prevents showing impossible options.
- **Alternatives Considered**:
  - *NVS Sentinel value (`255`)*: Requires BLE round-trips to read the key before hiding, and is vulnerable to stale NVS values on reflash without erase.

### Decision 2: Firmware Feature Bitmask Semantics
- In `_handleSettingsGetFeatures()`:
  - `RK_SETTINGS_FEATURE_BLE` is set if `#if defined(RK_ENABLE_BLE)`
  - `RK_SETTINGS_FEATURE_WIFI` is set if `#if defined(RK_ENABLE_WIFI)`
  - `RK_SETTINGS_FEATURE_CLOUD` is set if `#if defined(RK_ENABLE_CLOUD) && defined(RK_ENABLE_WIFI)`
- This makes feature flags strictly represent **capabilities** rather than transient connection state.

### Decision 3: Safety Guard in `_handleSettingsNvsRawWrite()`
- If a client sends `RK_SETTINGS_CMD_NVS_RAW_WRITE` for `rk_wifi_on` / `rk_sta_ssid` / `rk_sta_pwd` when `#ifndef RK_ENABLE_WIFI`, the firmware returns `RK_SETTINGS_RESP_NVS_RAW_STATUS` with status `RK_SETTINGS_NVS_ERROR`.
- If a client sends writes for `rk_ble_on` when `#ifndef RK_ENABLE_BLE`, the firmware returns `RK_SETTINGS_NVS_ERROR`.

## Risks / Trade-offs

- **[Risk] Existing apps reading `dp.hasWifi` when WiFi is compiled but disabled by NVS**:
  - `dp.hasWifi` indicates compile-time support, while `dp.connectedTransport == TransportType.wifi` indicates whether the active link is WiFi. This matches the existing convention for `hasFs` and `hasOta`.
