## Context

The RadioKit companion app flasher uses `flutter_esptool` (v0.1.4) bridged through `FlserialPortAdapter` to `flserial`. In production testing, several failure modes were identified:
1. `FlasherProvider.startFlashing()` ignores `_eraseAll`, skipping `ESP_ERASE_FLASH` (0xD0) and leaving existing NVS and LittleFS (0x310000) partitions intact.
2. `FirmwareBundle.buildMergedImage()` creates an expensive `0xFF`-padded array spanning from 0x0 to the highest partition offset, which transfers unnecessary data over USB serial.
3. `FlserialPortAdapter.write()` does not await Android USB method channel writes, leading to buffer overruns and false early completion.
4. Flutter package dependencies in `pubspec.yaml` need updating to ensure compatibility with modern Flutter/Dart runtimes.

## Goals / Non-Goals

**Goals:**
- Implement reliable full chip erase (`ESP_ERASE_FLASH`) when `eraseAll` is enabled.
- Implement multi-part sequential flashing by writing each partition part at its exact manifest offset.
- Implement accurate aggregate progress calculation across multi-part flashes.
- Ensure Android USB bulk transfers in `FlserialPortAdapter` are awaited synchronously.
- Upgrade Flutter dependencies across `pubspec.yaml` and verify build/test health.

**Non-Goals:**
- Replace the underlying `flutter_esptool` library with an external native daemon.
- Modify the ESP Web Tools `.zip` package format specification.

## Decisions

### Decision 1: Sequential Multi-Part Flashing with Offset Targeting
- **Approach**: Iterate through `bundle.parts` and invoke `_flashService.writeFlash()` for each part (`offset: part.offset, data: part.bytes`).
- **Rationale**: Avoids sending hundreds of kilobytes or megabytes of `0xFF` padding across slow serial links when partitions (e.g. bootloader @ 0x0, app @ 0x10000, LittleFS @ 0x310000) have large gaps between them.
- **Alternative Considered**: Unified contiguous binary (kept only as an optional helper or fallback).

### Decision 2: Dedicated Full Chip Erase Step
- **Approach**: When `_eraseAll` is true, call `await _flashService.eraseFlash()` (120s timeout) before flashing partition 0. Update the UI status to indicate chip erase in progress.
- **Rationale**: Completely wipes all flash sectors (NVS configuration, all OTA slots, LittleFS storage) to guarantee a clean first boot state.

### Decision 3: Await Android USB MethodChannel Writes
- **Approach**: In `FlserialPortAdapter.write()`, on Android invoke `await _flserialChannel.invokeMethod('writeUsbDevice', ...)` instead of unawaited `_serial.write()`.
- **Rationale**: Guarantees backpressure and prevents race conditions with SLIP framing and device responses.

### Decision 4: Dependency Modernization
- **Approach**: Bump outdated package dependencies in `pubspec.yaml` to their latest compatible stable releases and run `flutter pub upgrade`.

## Risks / Trade-offs

- **[Risk] Long chip erase duration causing user to think app is frozen** → **Mitigation**: Update UI status to `"Erasing flash memory (15–30s)..."` with active spinner and logging.
- **[Risk] Serial connection drop mid-part** → **Mitigation**: Log the exact failing partition and offset for rapid diagnostics and permit retries without app restart.
- **[Risk] Breaking API changes from package updates** → **Mitigation**: Run `flutter analyze` and `flutter test` after upgrading.
