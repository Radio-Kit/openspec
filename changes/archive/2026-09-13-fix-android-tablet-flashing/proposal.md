# Proposal: Fix Android Tablet ESP32 Flashing and Hardware Reboot

## Why

Flashing ESP32 firmware directly from the RadioKit Flutter companion app running on an Android tablet was failing while reporting a false "success" state in the UI. 

Investigation of the live Android runtime and ESP32 ROM bootloader protocol revealed three distinct failure points:
1. Android 14+ (SDK 34+) system USB permission broadcasts were silently dropped because the permission `BroadcastReceiver` was registered with `RECEIVER_NOT_EXPORTED`.
2. The flasher attempted compressed writes (`compress: true` / `FLASH_DEFL_BEGIN`) against the ESP32-S3 ROM bootloader, which only supports uncompressed writes (`FLASH_BEGIN`) unless an esptool RAM stub is executing.
3. Flash MD5 verification was disabled by default, causing silent write rejections to be reported as success, and no post-flash hardware reset was triggered to boot the chip into application mode.

Fixing these issues enables robust direct USB OTG flashing from Android tablets.

## What Changes

- **Android USB Permission Fix**: Register USB permission receivers with `Context.RECEIVER_EXPORTED` on Android 13+ (API 33+) with `FLAG_MUTABLE` PendingIntents so the system USB permission callback is reliably received.
- **ROM Direct Flasher Engine (`flasher_provider.dart`)**:
  - Configure `compress: false` on `FlashParameters` for ROM bootloader compatibility.
  - Configure `verify: true` to enforce cryptographic MD5 checksum verification against the ESP32 ROM.
  - Implement a post-flash reset sequence (clearing `RTC_CNTL_OPTION1_REG`, pulsing DTR/RTS, and recycling serial connections) so the target automatically boots the newly flashed application.
- **Firmware Verification**:
  - Build and flash `Filesystem_LED` release bundle to verify LittleFS bulk-FS protocol and GPIO 42 builtin LED on TrackLink V2.
  - Build and flash `BasicSwitch` release bundle to verify bidirectional widget synchronization on TrackLink V2.

## Capabilities

### New Capabilities
- `android-otg-flasher`: Reliable in-app USB OTG firmware flashing on modern Android versions (API 33+) with ROM bootloader fallback, MD5 verification, and post-flash reset.

### Modified Capabilities
- `flash-erase-policy`: Update flash verification and reset requirements to ensure full verification before declaring flashing complete.

## Impact

- `radiokit-app/lib/providers/flasher_provider.dart`: Update `FlashParameters` options and add post-flash hardware reset execution.
- `radiokit-app/android/...`: Update USB permission receiver registration flags.
- `flserial` Android plugin: Ensure USB permission intents and receivers support Android 14/15/16.
- `rk-arduino/examples/`: Updated target firmware binaries for TrackLink V2.
