# clean-slate-firmware-boot Specification

## Purpose
TBD - created by archiving change robust-flasher-and-device-onboarding. Update Purpose after archive.
## Requirements
### Requirement: Non-blocking example setup
Example sketches SHALL NOT use blocking loops waiting for USB CDC DTR signals.

#### Scenario: Standalone boot without USB host
- **WHEN** the ESP32 powers up standalone on battery or power adapter
- **THEN** setup executes immediately and initializes all configured transports including BLE

### Requirement: Self-formatting LittleFS partition
The firmware runtime SHALL initialize LittleFS with auto-format enabled (`LittleFS.begin(true)`), creating a clean filesystem on first boot if blank.

#### Scenario: Blank flash first boot
- **WHEN** an ESP32 boots with an unformatted LittleFS partition (0x310000)
- **THEN** LittleFS formats automatically and loads the compiled-in RADIOKIT.h layout

