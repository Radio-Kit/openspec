## Context

RadioKit builds firmware across multiple ESP32 target boards (e.g. `tracklink_v3`, `esp32s3`, `esp32`). Currently, the build scripts produce separate binaries:
- `firmware.factory.bin` (starting at offset `0x0000`, containing bootloader + partition table + otadata + app)
- `firmware.bin` (starting at offset `0x10000`, containing only the application code for OTA updates)

When users flash via the RadioKit Flutter app flasher page or remote API, allowing raw `.bin` files leads to bricked devices if an app `.bin` is flashed at offset `0x0000` (overwriting the bootloader and partition table). Furthermore, flashing over ESP32-S3 native USB-JTAG-Serial experiences connection drops when redundant DTR/RTS resets are issued.

To solve this, RadioKit standardizes on the open **ESP Web Tools `.zip` bundle format**.

## Goals / Non-Goals

**Goals:**
- Provide a single `.zip` firmware package containing `manifest.json` that serves both USB factory flashing and runtime OTA updates.
- Remove raw `.bin` file selection from the Flasher tab entirely (strict `.zip` requirement).
- Ingest `manifest.json` (ESP Web Tools schema) to dynamically flash each binary part at its declared byte offset.
- Verify `chipFamily` compatibility (e.g., `ESP32-S3`) between the bundle manifest and the connected physical chip before erasing/writing flash.
- Fix ESP32-S3 native USB connection drops during flashing by removing redundant bootloader resets when the transport is already connected and synchronized.
- Enable the OTA update service to ingest the same `.zip` bundle by automatically resolving the application partition (`offset 65536` / `0x10000`) and streaming it to the OTA engine.

**Non-Goals:**
- Retaining backward compatibility for raw `.bin` files in the Flasher tab (explicitly removed).
- Implementing `.elf` parsing or custom compiler toolchains inside Flutter.
- Modifying the underlying ESP32-to-Flutter RadioKit frame protocol.

## Decisions

### 1. Adopt Standard ESP Web Tools Manifest Schema in `.zip`
- **Choice**: Structure release bundles as standard `.zip` archives containing `manifest.json` and partition binaries.
  ```json
  {
    "name": "RadioKit RC Engine",
    "version": "1.0.0",
    "builds": [
      {
        "chipFamily": "ESP32-S3",
        "parts": [
          { "path": "bootloader.bin", "offset": 0 },
          { "path": "partitions.bin", "offset": 32768 },
          { "path": "boot_app0.bin",  "offset": 57344 },
          { "path": "firmware.bin",   "offset": 65536 },
          { "path": "littlefs.bin",   "offset": 3211264 }
        ]
      }
    ]
  }
  ```
- **Rationale**: ESP Web Tools is the de-facto standard across Web Serial flashers, Home Assistant, ESPHome, and Tasmota. Using this format ensures cross-compatibility with web flashers and third-party tools while utilizing Dart's built-in `archive` package.
- **Alternatives Considered**:
  - *Custom `.rkpkg` binary format*: Incompatible with external web tools.
  - *UF2 format*: 50% block overhead and lack of board/chip JSON metadata.

### 2. Strict Elimination of Raw `.bin` Files in Flasher
- **Choice**: File pickers and API endpoints in the Flasher tab exclusively accept `.zip` files containing `manifest.json`.
- **Rationale**: Eliminates user error where OTA binaries are inadvertently flashed at `0x0000`. Guarantees all required partitions (bootloader, partition table, otadata) are present during a factory flash.

### 3. Multi-Part Sequential Flash Execution
- **Choice**: `FlasherProvider` iterates over `manifest.parts` sorted by offset and executes `FlashService.writeFlash(FlashParameters(offset: part.offset, data: part.bytes))` for each partition.
- **Rationale**: Guarantees each component is written to its exact flash boundary without needing monolithic multi-megabyte padded files.

### 4. How the Same `.zip` Bundle Powers OTA Updates
- **Choice**: When a `.zip` bundle is selected in the OTA Update tab:
  1. The OTA service inspects `manifest.json`.
  2. It identifies the application partition (part where `offset == 65536` / `0x10000` or declared as the app payload).
  3. It extracts only `firmware.bin` from the archive and streams it directly to the device's OTA engine (`ota_0` or `ota_1` slot).
  4. If `littlefs.bin` is present and filesystem update is requested, it extracts and streams `littlefs.bin` to the storage partition.
- **Rationale**: Users and CI release pipelines only need to handle a single `.zip` asset per board version for both cable and wireless workflows.

### 5. Fix ESP32-S3 USB Connection Teardown
- **Choice**: In `FlasherProvider.startFlashing()`, remove the redundant call to `_enterBootloaderMode()` inside `_reprepareForFlashing()` when the connection is already active and synchronized.
- **Rationale**: On ESP32-S3 native USB-JTAG-Serial, toggling DTR/RTS resets the USB transceiver, causing the USB device to disconnect and re-enumerate with a new USB address.

## Risks / Trade-offs

- **[Risk] User attempts to upload old `.bin` file** → The parser immediately displays a clear, actionable error dialog explaining that a `.zip` bundle containing `manifest.json` is required.
- **[Risk] Chip Family Mismatch (e.g. flashing S3 bundle onto standard ESP32)** → The parser checks `detectedChip.family` against `build.chipFamily` and rejects the flash before any erase or write occurs.
- **[Risk] Missing partition in zip archive** → Strict validation of all paths listed in `manifest.json` against archive entries prior to beginning the flash.
