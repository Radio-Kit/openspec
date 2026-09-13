## ADDED Requirements

### Requirement: First-visit spotlight coach marks for primary workflows
The system SHALL present an in-situ spotlight tour overlay highlighting key actions on the first time a user views the Models Tab, Designer Screen, and Flasher Tab.

#### Scenario: User visits Models tab for the first time
- **WHEN** user opens the app on a fresh install and views the Models tab
- **THEN** an overlay highlights the Scan button, Pair action, and tab switcher sequentially, with step indicators and a dismiss button

#### Scenario: User visits Designer screen for the first time
- **WHEN** user launches the Designer screen and the canvas loads
- **THEN** an overlay highlights the widget toolbox, Inspector panel, Play Mode toggle, and C++ code export button

#### Scenario: User visits Flasher tab for the first time
- **WHEN** user switches to the Flasher tab
- **THEN** an overlay highlights the serial port selector, firmware catalog item, and Flash Firmware button

### Requirement: Tour persistence and manual reset
The system SHALL record tour completion in local settings so tours do not re-appear on subsequent screen visits, and SHALL provide a manual reset button in settings.

#### Scenario: Tour already seen on subsequent visits
- **WHEN** user returns to a screen whose tour was previously completed or dismissed
- **THEN** the screen displays normally without displaying the tour overlay

#### Scenario: User resets help guides in settings
- **WHEN** user navigates to the System/Settings screen and taps "Reset Help & Guides"
- **THEN** all tour completion flags are cleared, and tours display again upon visiting each respective screen
