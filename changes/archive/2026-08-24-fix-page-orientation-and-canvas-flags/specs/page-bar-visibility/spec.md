## MODIFIED Requirements

### Requirement: Persist page bar visibility in config
The page bar visibility states SHALL be stored in the JSON config under `canvas.showPageBar` and `canvas.showControlPageBar`, and transmitted over the wire protocol via `CONF_DATA` canvas flags. The default value SHALL be `true` when fields or wire flags are missing.

#### Scenario: Save visibility state
- **WHEN** user hides the page bar or control page bar in the designer
- **THEN** the JSON config contains `"showPageBar": false` or `"showControlPageBar": false` in the canvas object

#### Scenario: Load visibility state from wire
- **WHEN** the app receives CONF_DATA with canvasFlags bit 1 cleared
- **THEN** the control UI page switcher tab bar is hidden

#### Scenario: Backward compatibility
- **WHEN** a design config or CONF_DATA payload without canvasFlags is loaded
- **THEN** the page bar and control page bar default to visible
