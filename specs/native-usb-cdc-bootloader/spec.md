# native-usb-cdc-bootloader Specification

## Purpose
TBD - created by archiving change robust-flasher-and-device-onboarding. Update Purpose after archive.
## Requirements
### Requirement: Native USB CDC 1200-baud auto-reset touch
The flasher provider SHALL attempt a 1200-baud CDC touch combined with standard DTR/RTS pulsing to trigger ROM download mode on native USB CDC boards.

#### Scenario: Auto-reset native USB ESP32-S3
- **WHEN** connect is initiated on an ESP32-S3 native USB port
- **THEN** the flasher pulses DTR/RTS and attempts 1200-baud reset before synchronizing with the ROM bootloader

### Requirement: Manual bootloader entry guidance
If ROM synchronization fails after auto-reset attempts, the flasher SHALL display clear guidance explaining how to enter bootloader mode manually.

#### Scenario: Sync failure tip
- **WHEN** ROM synchronization times out
- **THEN** the console log displays a tip advising to hold BOOT and tap RESET

