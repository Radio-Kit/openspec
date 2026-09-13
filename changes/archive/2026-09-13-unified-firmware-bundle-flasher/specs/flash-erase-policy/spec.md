## ADDED Requirements

### Requirement: Multi-Part Sequential Flash Execution
The Flasher service SHALL iterate over all partition parts defined in `manifest.json` sorted by ascending offset and write each partition chunk to its designated flash address.

#### Scenario: Multi-part bundle flashing
- **WHEN** flashing is initiated for a bundle with bootloader, partitions, otadata, and app parts
- **THEN** each part is written to its specified flash offset with continuous progress reporting

### Requirement: Chip Family Compatibility Validation
The Flasher service SHALL verify that the detected physical ESP chip family matches the `chipFamily` declared in the bundle's `manifest.json` before writing to flash.

#### Scenario: Matching chip family
- **WHEN** flashing an ESP32-S3 bundle onto a connected ESP32-S3 device
- **THEN** validation passes and the flashing sequence proceeds

#### Scenario: Mismatched chip family
- **WHEN** flashing an ESP32-S3 bundle onto a connected ESP32 (standard) device
- **THEN** the flasher halts before writing, aborts the operation, and reports a chip mismatch error to the user
