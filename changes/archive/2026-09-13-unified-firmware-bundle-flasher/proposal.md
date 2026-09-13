## Why

Currently, RadioKit firmware releases produce two separate binaries: a merged factory binary (`.factory.bin`) for base offset `0x0` USB flashing, and an app-only binary (`-ota.bin`) for offset `0x10000` OTA updates. Manually selecting raw `.bin` files in the Flasher tab is error-prone—flashing an OTA binary to offset `0x0` corrupts the bootloader and partition table, rendering boards unbootable. Furthermore, raw binaries lack board, chip family, and offset metadata.

By adopting the industry-standard ESP Web Tools manifest bundle format (a standard `.zip` containing `manifest.json` and component binaries), RadioKit can distribute a single release package that seamlessly powers both full USB factory flashing and runtime OTA updates, while eliminating raw `.bin` selection errors.

## What Changes

- **Unified Release Bundle (`.zip`)**: Standardize release packaging on the ESP Web Tools `.zip` format containing `manifest.json` and component binaries (`bootloader.bin`, `partitions.bin`, `boot_app0.bin`, `firmware.bin`, and optional `littlefs.bin`).
- **BREAKING: Strict Bundle Requirement in Flasher**: Remove raw `.bin` file selection from the Flasher tab. The Flasher UI and API will exclusively accept `.zip` bundles with valid `manifest.json`.
- **Multi-Offset Flash Execution**: `FlasherProvider` reads all parts from `manifest.json` and flashes each part sequentially at its exact specified flash offset.
- **Hardware & Chip Safety Validation**: Validate that the connected chip family matches `manifest.json`'s `chipFamily` prior to flashing.
- **ESP32-S3 Native USB-JTAG-Serial Fix**: Eliminate redundant DTR/RTS resets in `startFlashing()` that cause USB peripheral drops on native USB chips.
- **Unified OTA Update Ingestion**: Enable the OTA update service to ingest the same `.zip` release bundle, automatically extracting and streaming the app partition (`offset 0x10000`) to the device's OTA slot (and optionally `littlefs.bin` to storage).

## Capabilities

### New Capabilities
- `unified-firmware-bundle`: Parsing and execution of ESP Web Tools compatible `.zip` firmware packages for both USB flashing and OTA update pipelines.

### Modified Capabilities
- `flash-erase-policy`: Update flash execution requirements to iterate multi-part manifests and validate chip family compatibility before writing.

## Impact

- `radiokit-app/lib/services/firmware_bundle_parser.dart`: New service to inspect, extract, and validate `.zip` bundles and `manifest.json`.
- `radiokit-app/lib/providers/flasher_provider.dart`: Restrict file picker to `.zip` bundles, flash multi-part segments, and remove redundant reset loops.
- `radiokit-app/lib/screens/flasher_screen.dart`: Update UI file picker labels and bundle metadata display.
- `radiokit-app/lib/services/remote_access_service.dart`: Update `/api/flasher/select-firmware` to accept `.zip` bundle data and reject raw `.bin` files.
- `radiokit-app/lib/services/fs_protocol_service.dart` / `device_provider.dart`: Support ingesting `.zip` bundle for OTA update by extracting the app partition.
- `scripts/package_release.py` / release tools: Automate generation of `.zip` bundle with standard `manifest.json`.
