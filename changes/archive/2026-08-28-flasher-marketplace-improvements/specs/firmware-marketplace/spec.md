## MODIFIED Requirements

### Requirement: Connected Chip Family Matching and Badging
The Flasher Tab marketplace view SHALL inspect the connected ESP32 chip family from `FlasherProvider` and highlight compatible binary assets, excluding OTA binaries from the flasher list.

#### Scenario: Connected board chip matches binary asset
- **WHEN** `FlasherProvider` is connected to an `ESP32-S3` board
- **THEN** release assets with `esp32s3` chip tokens display a green `[MATCHES YOUR BOARD]` badge and are selected by default.

#### Scenario: Connected board chip does not match binary asset
- **WHEN** `FlasherProvider` is connected to an `ESP32-S3` board and the release contains `esp32c3` assets
- **THEN** the `esp32c3` asset is dimmed with a `[CHIP MISMATCH]` warning badge, but remains selectable if overridden.

#### Scenario: Excluding OTA binaries from Flasher list
- **WHEN** release assets contain `*-ota.bin` or have flashType marked as `ota`
- **THEN** the USB Flasher marketplace view filters out OTA binaries and displays only full factory/bootloader flash images.

### Requirement: One-Click Stream and Flash Integration
The companion app SHALL download selected marketplace binaries into memory and stage them into `FlasherProvider`.

#### Scenario: User triggers select firmware on a marketplace asset
- **WHEN** the user clicks "SELECT FIRMWARE" on a selected binary asset in the Flasher Tab
- **THEN** the app streams the `.bin` bytes over HTTP with a progress indicator
- **AND** passes the bytes directly to `FlasherProvider.setSelectedFirmwareDirect()` to stage it for the user to flash via the "FLASH FIRMWARE" button.

## ADDED Requirements

### Requirement: Firmware Asset Card Formatting and Staging
The Flasher Tab marketplace view SHALL display repository cards collapsed by default, and render available binaries as human-readable cards with Board, Chip, and Variant tags.

#### Scenario: Repository cards render collapsed on load
- **WHEN** the user opens the Flasher Tab
- **THEN** all repository cards are collapsed by default and only expand when explicitly clicked.

#### Scenario: Binary assets render with human-readable metadata
- **WHEN** a release contains parsed binary assets (e.g. `RC_Engine-v1.0.0-esp32s3-GTRACK.bin`)
- **THEN** the card renders `GTRACK` as the primary title, displays `Chip: ESP32S3` and Variant tag (if present), file size, and the subtle asset filename.
