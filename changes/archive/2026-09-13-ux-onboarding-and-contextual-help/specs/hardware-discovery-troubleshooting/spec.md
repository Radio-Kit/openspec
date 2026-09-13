## ADDED Requirements

### Requirement: Actionable hardware discovery empty states
The system SHALL present an actionable diagnostic checklist when device scanning or serial port scanning detects no physical hardware.

#### Scenario: No devices found during scan on Models tab
- **WHEN** a scan finishes with zero discovered devices and no paired devices connected
- **THEN** the screen displays a hardware checklist verifying Bluetooth power, Location/Bluetooth app permissions, USB-OTG connection, and microcontroller power

#### Scenario: No serial ports detected on Flasher tab
- **WHEN** serial port scanning returns an empty list
- **THEN** the Flasher tab displays an empty state with verification steps for USB data cable integrity, driver requirements (CH340/CP2102), and OS permission prompts

### Requirement: Interactive manual bootloader helper
The system SHALL provide an interactive visual diagram demonstrating the manual button timing sequence for entering the ESP32 download mode.

#### Scenario: Flasher fails to trigger download mode or user requests bootloader guide
- **WHEN** user taps "Bootloader Guide" or auto-reset fails during flashing
- **THEN** an illustrated modal displays the step-by-step procedure: hold BOOT (GPIO0), press and release EN (Reset), then release BOOT
