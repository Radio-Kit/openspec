## MODIFIED Requirements

### Requirement: Mandatory Flash Erase on Firmware Upload
The documentation and agent guidelines SHALL explicitly specify that all firmware upload procedures (via CLI, PlatformIO, OTA, or web/app flasher) include the flash erase option (`-erase`, `--erase-flash`, or `eraseAll: true`) unless the user explicitly requests preserving existing device settings, and in-app flashing SHALL enforce checksum verification across erased and written regions.

#### Scenario: Agent or developer builds and uploads firmware
- **WHEN** an agent or developer uploads code to an ESP32 microcontroller
- **THEN** the upload command or API call includes the flash erase flag (`-erase` / `eraseAll: true`) to ensure stale NVS configuration (old BLE device name, stale WiFi credentials) is cleared before first boot

#### Scenario: User requests preserving settings across upload
- **WHEN** the user explicitly specifies that previous device settings must be preserved
- **THEN** the upload command is run without the flash erase flag
