## MODIFIED Requirements

### Requirement: Control screen orientation re-lock
When the user switches pages in the control screen or when the initial connection handshake processes CONF_DATA, the phone's system orientation SHALL be locked to match the active page's effective orientation.

#### Scenario: Initial connection applies active page orientation
- **WHEN** the app connects to a device and receives CONF_DATA with activePage = N
- **THEN** the phone's system orientation is locked to the effective orientation of page N (_pageOrientations[N])

#### Scenario: Page switch updates phone orientation
- **WHEN** user switches from a landscape page to a portrait page in the control screen
- **THEN** the phone's system orientation is locked to portrait

#### Scenario: Same orientation no-op
- **WHEN** user switches between two pages with the same effective orientation
- **THEN** the phone's system orientation remains unchanged
