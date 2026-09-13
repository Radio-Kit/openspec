## ADDED Requirements

### Requirement: Standard ESP Web Tools Manifest Bundle
The release packaging pipeline and companion application SHALL support firmware release archives in standard `.zip` format containing a valid `manifest.json` following the ESP Web Tools schema.

#### Scenario: Valid ESP Web Tools bundle parsed
- **WHEN** a `.zip` file containing `manifest.json` and referenced binary parts is selected
- **THEN** the application parses the project metadata, builds list, target `chipFamily`, and part offsets/paths without errors

#### Scenario: Malformed or missing manifest rejected
- **WHEN** a `.zip` archive missing `manifest.json` or containing invalid JSON is selected
- **THEN** the application rejects the package and displays an error indicating the bundle is invalid

### Requirement: Strict Bundle Enforcement in Flasher
The Flasher tab and flasher API endpoints SHALL strictly require `.zip` firmware bundles and SHALL reject raw `.bin` files.

#### Scenario: User selects a raw .bin file in flasher
- **WHEN** a user attempts to select a `.bin` file in the Flasher UI or API
- **THEN** the system rejects the file and prompts the user to supply a `.zip` firmware bundle

#### Scenario: User selects a valid .zip bundle in flasher
- **WHEN** a user selects a valid `.zip` bundle containing `manifest.json`
- **THEN** the Flasher UI displays the bundle name, version, target chip family, and list of partition parts

### Requirement: Unified OTA Ingestion from Bundle
The OTA update service SHALL accept the same standard `.zip` firmware bundle, automatically extracting the application binary corresponding to flash offset `65536` (`0x10000`).

#### Scenario: User provides .zip bundle for OTA update
- **WHEN** a user initiates an OTA update with a `.zip` firmware bundle
- **THEN** the OTA engine extracts the application binary part (`offset 65536`) and streams it directly to the active device's inactive OTA slot

#### Scenario: Dual application and filesystem update via bundle
- **WHEN** a `.zip` bundle includes both an app part and a filesystem part (`littlefs.bin`)
- **THEN** the update pipeline allows streaming both application and storage updates from the single package
