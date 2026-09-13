## Context

Hardware testing on ESP32-S3 boards with Native USB CDC (such as the Mikro board) revealed practical obstacles in the flashing and device onboarding pipeline:
- Native USB CDC boards do not enter ROM download mode via traditional DTR/RTS transistor toggling unless accompanied by a 1200-baud touch.
- Previous LittleFS configurations at partition `0x310000` overrode freshly flashed sketches unless wiped, and opcode `0xD0` (full chip erase) hangs on ESP32-S3 ROM bootloaders without a RAM stub.
- Example sketches like `BasicSwitch` blocked in `setup()` with `while (!Serial)` if no host opened the USB CDC COM port.
- Android 12+ BLE scanning required Location Services and runtime permissions, combined with a flexible BLE name matching policy.

## Goals / Non-Goals

**Goals:**
- Provide single-pass unified flashing that streams the entire multi-part firmware bundle from `0x0000` with 0xFF padding.
- Support native USB CDC auto-reset via 1200-baud touch + DTR/RTS pulsing.
- Support clean-slate boot where Arduino formats LittleFS automatically (`LittleFS.begin(true)`), eliminating the need for `littlefs.bin` packaging.
- Clean up example sketches so they boot and run standalone on battery/power supply without blocking on USB CDC ports.
- Ensure Android Bluetooth and Location runtime permissions are requested silently on app startup and BLE name filtering accepts non-prefixed names.

**Non-Goals:**
- Packaging `littlefs.bin` into every firmware bundle (unnecessary because Arduino runtime formats automatically on blank flash).
- Supporting legacy ESP8266 proprietary protocols.

## Decisions

### Decision 1: Single-Pass Unified Compressed Flashing
- **Rationale**: `flutter_esptool` decompressor session terminates after writing Part 1 if called in a loop. Merging all parts into a contiguous image (`0x0000` to end of app) with 0xFF padding compresses down to negligible size and writes in a single, robust session.
- **Alternative considered**: Multiple disconnect/re-connect cycles between partition parts (slow and error-prone).

### Decision 2: 1200-Baud Native USB CDC Auto-Reset
- **Rationale**: When ESP32-S3 runs Arduino firmware with USB CDC enabled, opening at 1200 baud triggers the CDC driver to jump into ROM download mode.
- **Alternative considered**: Forcing the user to manually press BOOT/RESET every time (poor UX).

### Decision 3: Remove Blocking `while (!Serial)` in Examples
- **Rationale**: On ESP32-S3 Native USB, `while (!Serial)` blocks indefinitely unless connected to a desktop serial monitor, preventing BLE initialization.
- **Alternative considered**: Short timeout `while (!Serial && millis() < 2000)` (unnecessary delay).

## Risks / Trade-offs

- [Risk] ESP32-S3 running corrupted/hung firmware cannot handle 1200-baud touch → Mitigation: Flasher provider displays actionable log tip: "Hold BOOT, tap RESET, release BOOT".
- [Risk] Old Android versions require explicit Location toggle → Mitigation: Silent permission request on startup with clear error status if Bluetooth is off.
