## MODIFIED Requirements

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
