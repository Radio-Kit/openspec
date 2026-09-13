## Why

End-to-end hardware testing on ESP32-S3 boards (such as the Mikro board) revealed three critical pain points during flashing and onboarding:
1. Native USB CDC chips do not use standard dual-transistor auto-reset circuits and need 1200-baud CDC touches combined with DTR/RTS pulsing to enter ROM bootloader mode without manual button intervention.
2. When flashing a new example sketch (like `BasicSwitch`), pre-existing LittleFS `/config.json` files on flash partition `0x310000` overrode the newly compiled `RADIOKIT.h`, and examples containing `while (!Serial)` locked the CPU permanently on standalone boot.
3. Android BLE scanning was blocked by strict `RK_` name matching and system Location/Bluetooth permission handling.

This change establishes a robust, single-pass unified flasher, clean-slate LittleFS self-initialization, non-blocking example sketches, and automatic BLE discovery.

## What Changes

- **Single-Pass Unified Flashing**: Package and stream full multi-part firmware bundles as a unified, compressed 0xFF-padded binary image starting at `0x0000`. Eliminates deflate termination failures across sequential partition writes.
- **Native USB CDC Auto-Reset**: Implement a 1200-baud touch sequence in `FlserialPortAdapter` and `FlasherProvider._enterBootloaderMode()` to trip the ESP32-S3 USB CDC ROM into download mode, with actionable user guidance if manual button entry is needed.
- **Clean-Slate LittleFS Initialization**: Ensure Arduino runtime self-formats LittleFS partitions on first boot if unformatted (`LittleFS.begin(true)`), avoiding the need for heavy `littlefs.bin` packaging. Remove blocking `while (!Serial)` loops from all example sketches.
- **Automatic BLE Discovery**: Relax `UniversalBle.onScanResult` filter to discover devices with or without `RK_` prefixes and request Android Bluetooth/Location permissions on startup.

## Capabilities

### New Capabilities
- `native-usb-cdc-bootloader`: 1200-baud touch and DTR/RTS auto-reset protocol tailored for native USB CDC microcontrollers (ESP32-S3, ESP32-C3/C6).
- `clean-slate-firmware-boot`: Non-blocking setup execution in examples and runtime self-formatting for clean LittleFS initialization.

### Modified Capabilities
- `flash-erase-policy`: Update flashing pipeline to use unified compressed binary streaming and avoid unsupported `0xD0` chip erase timeouts.
- `ble-scan-lifecycle`: Allow flexible advertising name filtering and silent permission handling for reliable device pairing.

## Impact

- **Flutter Companion App** (`radiokit-app`):
  - Updated `FlasherProvider` (`_enterBootloaderMode()`, `startFlashing()`).
  - Updated `BleService` (`startScan()`).
  - Dependencies: `flutter_esptool: ^0.1.5`, `flutter_secure_storage: 10.3.1`, `re_editor: ^0.10.0`.
- **Arduino Library & Examples** (`rk-arduino`):
  - `BasicSwitch` and example sketches updated to remove blocking `while (!Serial)` waits.
