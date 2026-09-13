## Why

Firmware flashing in the RadioKit companion app currently experiences reliability and configuration persistence issues:
1. When users check "Erase Flash", `flasher_provider.dart` does not execute an actual full chip erase command (`ESP_ERASE_FLASH`), leaving stale NVS settings, OTA slots, and existing LittleFS partitions (`0x310000`) untouched.
2. The flasher builds a single merged binary filled with `0xFF` padding between offsets, which transmits large empty blocks over serial and fails if partitions are sparsely laid out or when flashing multi-part bundles.
3. On Android, `FlserialPortAdapter.write()` does not await the underlying USB bulk transfer completion, causing writes to return prematurely before physical SPI flash programming finishes.
4. Flutter dependencies in `radiokit-app` require updating to their latest compatible versions.

## What Changes

- **Explicit Full Chip Erase**: Implement `_flashService.eraseFlash()` in `FlasherProvider.startFlashing()` when `eraseAll` is enabled (with proper 120s timeout and UI status updates).
- **Multi-Part Flashing Pipeline**: Write each firmware partition part (`bootloader`, `partitions`, `otadata`, `app`, `littlefs`) independently at its specified manifest offset, computing aggregate multi-part progress, with a fallback to unified merged image if needed.
- **Synchronous Android USB Transport**: Ensure `FlserialPortAdapter` properly awaits USB bulk writes and avoids dropping device connection during baud rate transitions.
- **Flutter Package Updates**: Update `radiokit-app` dependencies to their latest compatible versions and run `flutter pub upgrade`.

## Capabilities

### New Capabilities
- `multi-part-flashing`: Sequential independent partition writes for ESP32 firmware bundles with aggregate progress tracking and robust full chip erase handling.

### Modified Capabilities
- `flash-erase-policy`: Update requirements to enforce that `FlasherProvider` executes `ESP_ERASE_FLASH` (0xD0) when `eraseAll: true` before writing partition binaries.

## Impact

- `radiokit-app/lib/providers/flasher_provider.dart`: `startFlashing()` implementation updated to call `eraseFlash()` and flash parts sequentially.
- `radiokit-app/lib/services/flserial_port_adapter.dart`: Write flow control and Android USB synchronization fixes.
- `radiokit-app/pubspec.yaml`: Package dependency version bumps and resolution.
