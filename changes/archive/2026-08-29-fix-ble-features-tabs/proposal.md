## Why

When connecting to a device via BLE, the config bottom sheet is missing OTA (Firmware) and FS (Filesystem) tabs even though the device supports these features. The root cause is a combination of silent failure in the features request path and a 2-second timeout that is too aggressive for concurrent BLE traffic. Additionally, when a user authenticates with a user-level password, FS/OTA tabs are hidden by design but there is no UI indication explaining why.

The designer UI is also missing password fields: the user password field is entirely absent from the designer inspector and codegen, and the device password field is hidden when empty (preventing users from setting a password for the first time).

## What Changes

- **Add diagnostic logging** to `_requestFeatures()` so write failures, timeouts, and response values are visible in the console. Currently all errors are silently swallowed with bare `catch (_)`.
- **Add retry logic** to `_requestFeatures()` — if the first attempt times out (2s), retry once after a short delay. Five concurrent fire-and-forget requests can overwhelm the firmware's TX queue on BLE.
- **Add UI indicator** when `isUserMode` hides FS/OTA tabs — show a lock icon or tooltip explaining that device-level authentication is required for these tabs, so users know why they're missing.
- **Fix missing password fields in designer** — add device password and user password fields to the MODEL section of the designer inspector. User password field is only visible when device password is non-empty. Make the device password field always visible.
- **Move remote link fields to Features section** — move FS URL and OTA URL fields from the standalone REMOTE LINKS section into the FEATURES section, conditionally shown under their respective feature toggles (FS URL under "Enable Filesystem", OTA URL under "Enable OTA").
- **Fix double `startBLE()` in generated firmware** — the codegen template calls `RadioKit.startBLE()` in both `initRadioKit()` and `setup()`, causing a NimBLE reinitialization during boot. Remove the duplicate call.

## Capabilities

### New Capabilities

- `ble-features-diagnostic`: Diagnostic logging and retry logic for the BLE features request path, ensuring the features bitmask is reliably received and any failures are visible.
- `auth-tab-gating-indicator`: UI indicator in the config bottom sheet explaining why FS/OTA tabs are hidden when in user-level auth mode.
- `designer-password-fields`: Add device password and user password fields to the MODEL section; user password visible only when device password is non-empty.
- `designer-features-links`: Move FS URL and OTA URL from REMOTE LINKS section into FEATURES section, conditionally shown under their respective feature toggles.

### Modified Capabilities

- `ota-runtime-enablement`: Existing spec covers `enableOTA()` and feature reporting. The features bitmask reliability fix ensures this contract is honored over BLE transport.

## Impact

- **Flutter app (`device_provider.dart`)**: `_requestFeatures()` method — add logging, retry, and timeout handling.
- **Flutter app (`models_tab.dart`)**: `_DeviceInfoTabs` — add lock icon/tooltip when `isUserMode` hides FS/OTA tabs.
- **Flutter designer (`designer_inspector.dart`)**: Add device password and user password fields to MODEL section; move FS/OTA URL fields into FEATURES section under toggles.
- **Flutter designer (`designer_state.dart`)**: Add `userPassword` state field and serialization.
- **Flutter designer codegen (`json_arduino_generator.dart`)**: Generate `RadioKit.config.user_password = "..."` when set.
- **Firmware codegen (`RADIOKIT.h` template)**: Remove duplicate `startBLE()` call from generated `initRadioKit()` or `setup()`.
- **Existing specs**: `ota-runtime-enablement` — the feature reporting contract is unchanged, but reliability is improved.
