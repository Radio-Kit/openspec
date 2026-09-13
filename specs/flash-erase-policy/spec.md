# flash-erase-policy Specification

## Purpose
Enforce mandatory flash erase policy during firmware upload to clear stale ESP32 NVS settings and document the rationale.
## Requirements
### Requirement: Mandatory Flash Erase on Firmware Upload
The documentation, agent guidelines, and flasher implementation SHALL ensure that when full flash erase is enabled (`eraseAll: true`), the flasher executes a full chip erase command (`ESP_ERASE_FLASH` opcode `0xD0`) with a 120-second timeout prior to writing any partition data, clearing all NVS settings, OTA slots, and existing LittleFS data.

#### Scenario: Agent or developer builds and uploads firmware
- **WHEN** an agent or developer uploads code to an ESP32 microcontroller
- **THEN** the upload command or API call includes the flash erase flag (`-erase` / `eraseAll: true`) to ensure stale NVS configuration (old BLE device name, stale WiFi credentials) is cleared before first boot

#### Scenario: User requests preserving settings across upload
- **WHEN** the user explicitly specifies that previous device settings must be preserved or unchecks erase all
- **THEN** the upload command is run without full chip erase

#### Scenario: User flashes with erase all checked in companion app
- **WHEN** the user initiates flashing with `eraseAll == true` in the Flasher tab
- **THEN** `FlasherProvider` invokes `_flashService.eraseFlash()` and waits for complete chip erase before starting partition writes

### Requirement: Document NVS Flash Persistence Rationale
The firmware, OTA, remote, and example skills SHALL document the rationale for flash erasing: ESP32 NVS sector stores configuration parameters across standard flashes, which causes the microcontroller to advertise under previous BLE names or retain legacy transport credentials unless explicitly erased.

#### Scenario: Agent reviews skill documentation before flashing
- **WHEN** an agent reads `SKILLS/radiokit-firmware/SKILL.md`, `radiokit-ota`, `radiokit-remote`, or `.agents/skills/radiokit-example`
- **THEN** the skill provides a clear warning and visual/text explanation of why skipping flash erase leads to stale device name and config issues

### Requirement: Unified single-pass compressed flashing
The flasher provider SHALL build a single contiguous binary image with 0xFF padding across all bundle partition parts and stream it in one compressed write operation.

#### Scenario: Single-pass flash write
- **WHEN** the user initiates flashing for a firmware release bundle
- **THEN** all partition parts from offset 0x0000 are written in a single uninterrupted deflate session

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

