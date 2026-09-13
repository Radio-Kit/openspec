## ADDED Requirements

### Requirement: Android USB Permission Broadcast Reception
The application and serial plugins SHALL register USB permission receivers with `RECEIVER_EXPORTED` on Android 13+ (API 33+) and use `FLAG_MUTABLE` PendingIntents so system `UsbManager` permission results are delivered.

#### Scenario: User grants USB permission on modern Android
- **WHEN** the application requests USB permission for a connected ESP32 device on Android 14+
- **THEN** the system permission broadcast is delivered to the receiver and the port opens successfully

### Requirement: Direct ROM Bootloader Flashing Compatibility
The flashing engine SHALL use uncompressed writes (`compress: false`) and enable cryptographic verification (`verify: true`) when flashing ESP32 chips in ROM bootloader mode.

#### Scenario: Flashing firmware to ESP32-S3 ROM
- **WHEN** the flasher writes a multi-part or unified firmware image to an ESP32-S3
- **THEN** it sends uncompressed `FLASH_BEGIN` packets and validates the MD5 checksum of written sectors against device ROM

### Requirement: Post-Flash Hardware Reset
The flashing provider SHALL execute a post-flash hardware reset sequence using RTS/DTR lines to reboot the ESP32 out of download mode into user application mode.

#### Scenario: Flashing completes successfully
- **WHEN** the flash write and MD5 verification succeed
- **THEN** the provider strobes reset lines, clears download mode, and disconnects the flasher transport so the application runs
