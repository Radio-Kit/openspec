# contextual-help-system Specification

## Purpose
TBD - created by archiving change ux-onboarding-and-contextual-help. Update Purpose after archive.
## Requirements
### Requirement: Reusable contextual help badge
The system SHALL provide a standardized `HelpBadge` UI component displayed beside technical settings and labels across the application.

#### Scenario: User taps a help badge
- **WHEN** user taps a `HelpBadge` beside a technical setting (e.g. Baud Rate, CDC Touch, or LittleFS)
- **THEN** the system displays a modal bottom sheet containing a plain-language summary, the hardware rationale, and relevant troubleshooting guidance

### Requirement: Standardized help content definitions
The system SHALL provide structured help topic entries covering core embedded concepts including baud rates, native CDC bootloader behavior, LittleFS partitions, transport types (BLE, Serial, WiFi), and Ed25519 pairing credentials.

#### Scenario: User views help sheet for CDC Touch
- **WHEN** user taps the help badge beside CDC Touch mode
- **THEN** the help sheet explains the 1200-baud DTR/RTS pulsing sequence used to enter the ROM bootloader on native USB ESP32 chips

