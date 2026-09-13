## Context

The RadioKit companion application running on Android tablets allows users to flash firmware releases over USB OTG to connected ESP32 boards. During live testing on Android 14+ (SDK 36) with an ESP32-S3 (TrackLink V2), flashing attempts failed silently while the UI reported success.

Analysis of the system showed:
1. Android 14+ strict security policies drop broadcasts sent to `RECEIVER_NOT_EXPORTED` receivers when originated by system services like `UsbManager`.
2. `flutter_esptool` connects directly to the ESP32 ROM bootloader without an esptool RAM stub. The ESP32-S3 ROM bootloader does not support compressed deflate transfers (`FLASH_DEFL_BEGIN`), only uncompressed `FLASH_BEGIN`.
3. Flash MD5 verification was disabled by default (`verify: false`), masking write failures as successes.
4. No post-flash hardware reset was performed to transition the ESP32 out of download mode.

## Goals / Non-Goals

**Goals:**
- Enable reliable USB OTG permission granting on Android 13/14/15/16.
- Configure `flasher_provider.dart` for direct ESP32 ROM flashing with uncompressed packets and mandatory MD5 verification.
- Add post-flash hardware reset execution via DTR/RTS and port cycling.
- Build and verify firmware bundles for `Filesystem_LED` and `BasicSwitch` on TrackLink V2.

**Non-Goals:**
- Full custom RAM stub bundling for flutter_esptool (direct ROM flashing at 115200/460800 baud is fully functional).

## Decisions

1. **Register USB Permission Receivers with `RECEIVER_EXPORTED` on API 33+**:
   - Rationale: System server `UsbManager` runs outside the app's process and cannot dispatch broadcasts to `RECEIVER_NOT_EXPORTED` receivers on Android 14+.
   - Alternatives: Activity-result based USB permission (not supported by Android UsbManager, which relies on PendingIntent broadcast).

2. **Uncompressed Writes (`compress: false`) for ESP32-S3 ROM**:
   - Rationale: ESP32-S3 ROM bootloader natively supports `FLASH_BEGIN` (0x02) and `FLASH_DATA` (0x03). `FLASH_DEFL_BEGIN` (0x10) requires the esptool RAM stub.
   - Alternatives: Embedding the ESP32-S3 stub ELF binaries into Flutter assets (adds complexity and binary overhead).

3. **Mandatory Flash MD5 Verification (`verify: true`)**:
   - Rationale: Guaranteed validation of written flash against ROM computed MD5 prevents false positives.

4. **Hardware Reset Post-Flash**:
   - Rationale: ESP32-S3 stays in download boot mode until reset. Toggling RTS/DTR lines and re-opening serial transport restores normal boot execution.

## Risks / Trade-offs

- [Uncompressed flashing speed] → Flashing an uncompressed 700KB binary over USB CDC at 115200/460800 takes ~5-10 seconds, which is fast and completely reliable.
