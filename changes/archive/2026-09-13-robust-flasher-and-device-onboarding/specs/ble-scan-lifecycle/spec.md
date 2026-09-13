## ADDED Requirements

### Requirement: Flexible BLE advertising name discovery
The BLE service SHALL accept discovered peripherals with or without the standard `RK_` name prefix.

#### Scenario: Peripheral discovered without prefix
- **WHEN** a RadioKit device advertises with a custom name lacking the `RK_` prefix
- **THEN** the BLE service discovers the device and displays it in the pairing list
