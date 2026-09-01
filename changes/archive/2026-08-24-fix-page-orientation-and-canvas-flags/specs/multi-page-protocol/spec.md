## ADDED Requirements

### Requirement: Canvas flags in CONF_DATA
The protocol SHALL include a 1-byte canvas flags field in the CMD_CONF_DATA (0x02) packet payload positioned after the per-page orientations array. Bit 0 SHALL indicate showPageBar and bit 1 SHALL indicate showControlPageBar.

#### Scenario: CONF_DATA includes canvas flags
- **WHEN** the MCU transmits CONF_DATA for a multi-page configuration
- **THEN** the payload includes the canvasFlags byte after the pageOrientations array

#### Scenario: Default canvas flags fallback
- **WHEN** the app receives a legacy CONF_DATA payload lacking the canvasFlags byte
- **THEN** the app defaults canvasFlags to 0x03 (both showPageBar and showControlPageBar enabled)
