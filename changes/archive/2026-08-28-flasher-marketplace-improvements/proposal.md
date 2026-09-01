## Why

The Flasher Tab firmware marketplace view currently expands all repository cards by default, causing unnecessary vertical scrolling when multiple repositories are added. Furthermore, raw asset filenames (e.g. `RC_Engine-v1.0.0-esp32s3-GTRACK-factory.bin`) make identifying board and variant options difficult for users, and single-partition OTA binaries (`*-ota.bin`) clutter the USB flashing list. Finally, clicking "DOWNLOAD & FLASH" immediately triggers flashing rather than loading the binary into the main FIRMWARE section for user inspection and explicit flashing.

## What Changes

- **Collapsed Repository Cards by Default**: All repository cards in the Marketplace section render collapsed on initial load.
- **Filter Out OTA Binaries in Flasher**: Exclude `*-ota.bin` (single partition update binaries) from the USB Flasher binary list.
- **Human-Readable Firmware Item Cards**: Display firmware assets using clean structured metadata:
  - Board Name as the primary title (e.g., `GTRACK`)
  - Chip identifier tag (e.g., `ESP32S3`)
  - Variant tag when present (e.g., `CAN`, `4WD`)
  - File size and hardware compatibility badge (`MATCHES ESP32-S3`)
  - Subtle secondary display of the raw `.bin` asset filename
- **"SELECT FIRMWARE" Action**: Rename "DOWNLOAD & FLASH" to "SELECT FIRMWARE". When clicked, it downloads the chosen binary and stages it in the `FIRMWARE` section (`setSelectedFirmwareDirect`), allowing the user to inspect it and initiate flashing via the primary "FLASH FIRMWARE" button.
- **Remove `-factory` Suffix in `RC_brain`**: Update `RC_brain/scripts/package_release.py` so standard complete flash binaries are named `RC_Engine-<version>-<chip>-<env>.bin` without redundant `-factory`.

## Capabilities

### New Capabilities

### Modified Capabilities
- `firmware-marketplace`: Update repository display behavior (collapsed by default), filter out OTA binaries from USB flasher list, format binaries as human-readable Board/Chip/Variant cards, and change action to "SELECT FIRMWARE" staging.

## Impact

- `radiokit-app/lib/screens/home/flasher_tab.dart`: Updates `_MarketplaceSection` rendering, card expansion, binary filtering, and select firmware handler.
- `radiokit-app/lib/services/firmware_marketplace_service.dart`: Adds dedicated `board`, `variant`, and `displayName` extraction.
- `RC_brain/scripts/package_release.py`: Removes `-factory` suffix from release asset names.
