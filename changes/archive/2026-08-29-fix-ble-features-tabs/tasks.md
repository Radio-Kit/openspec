## 1. Diagnostic Logging for Features Request

- [x] 1.1 Add warning log in `_requestFeatures()` when `_writePacket()` throws (catch block with `catch (e)`)
- [x] 1.2 Add warning log in `_requestFeatures()` when the completer times out (catch `TimeoutException`)
- [x] 1.3 Add warning log in `_requestFeatures()` when the completer completes with an error (generic catch)
- [x] 1.4 Verify existing success log in `_handleSettingsFeaturesData()` includes bitmask hex value and all capability flags

## 2. Features Request Retry Logic

- [x] 2.1 Refactor `_requestFeatures()` to accept an optional attempt counter parameter (default 0, max 1 retry)
- [x] 2.2 Increase per-attempt timeout from 2s to 3s
- [x] 2.3 Add 300ms delay before retry when first attempt times out
- [x] 2.4 After retry (or success), complete the features completer with the final bitmask value

## 3. Auth Tab Gating Indicator

- [x] 3.1 In `_DeviceInfoTabsState.build()`, detect when `isUserMode && (hasFs || hasOta)` after the tab list is built
- [x] 3.2 Render a lock icon (`Icons.lock_outline`) next to the TabBar when the condition is true
- [x] 3.3 Wrap the lock icon in a `Tooltip` widget with message "Device-level access required for FS/OTA"
- [x] 3.4 Style the lock icon to match the tab bar's color scheme (use `context.tokens.onSurface` with alpha)

## 4. Designer Password Fields in MODEL Section

- [x] 4.1 Add `userPassword` field to `DesignerState` class in `flutter-widgets/lib/src/models/designer_state.dart`
- [x] 4.2 Add `setUserPassword(String value)` method to `DesignerState`
- [x] 4.3 Update `DesignerState.fromJson()` to read `config.user_password`
- [x] 4.4 Update `DesignerState.toJson()` to write `config.user_password`
- [x] 4.5 Update header file parser (`codegen/header_file_parser.dart`) to read `userPassword` from JSON
- [x] 4.6 In `designer_inspector.dart`, remove the `if (connectionPassword.isNotEmpty)` guard so device password field is always visible in MODEL section
- [x] 4.7 In `designer_inspector.dart`, add a user password field below the device password field, only visible when `connectionPassword.isNotEmpty`
- [x] 4.8 Update codegen (`codegen/json_arduino_generator.dart`) to generate `RadioKit.config.user_password = "..."` when non-empty

## 5. Move Remote Links to Features Section

- [x] 5.1 In `designer_inspector.dart`, remove the `_buildLinksSection(tokens)` call and the entire `_buildLinksSection` method
- [x] 5.2 In the FEATURES section, after the "Enable OTA" toggle, add a conditional FS URL text field (visible when `featureOta` is true)
- [x] 5.3 In the FEATURES section, after the "Enable Filesystem" toggle, add a conditional OTA URL text field (visible when `featureFilesystem` is true)
- [x] 5.4 Verify FS URL and OTA URL values are preserved when toggles are switched off and back on

## 6. Firmware Codegen Fix

- [x] 6.1 Locate the `RADIOKIT.h` codegen template that generates the `initRadioKit()` function
- [x] 6.2 Remove the `RadioKit.startBLE()` call from the generated `initRadioKit()` function body
- [x] 6.3 Verify that `initRadioKit()` still calls `RadioKit.begin()`, `RadioKit.startSerial(Serial)`, `RadioKit.enableFS()`, and `RadioKit.enableOTA()` (if applicable)
- [x] 6.4 Verify that the generated `.ino` file's `setup()` function still calls `RadioKit.startBLE()` after `initRadioKit()`

## 7. Verification

- [x] 7.1 Run Flutter analyzer to check for type errors in modified files
- [x] 7.2 Manually test: connect to a BLE device with OTA+FS features — verify tabs appear
- [x] 7.3 Manually test: connect with user-level password — verify lock icon appears and tabs are hidden
- [x] 7.4 Manually test: simulate features timeout — verify retry log messages appear in console
- [x] 7.5 Manually test: open designer — verify device password field is always visible in MODEL section
- [x] 7.6 Manually test: set device password in designer — verify user password field appears below it
- [x] 7.7 Manually test: toggle features on/off — verify FS/OTA URL fields appear/disappear correctly
- [x] 7.8 Manually test: set user password in designer — verify codegen output includes `user_password`
