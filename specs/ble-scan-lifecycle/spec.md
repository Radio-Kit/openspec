# ble-scan-lifecycle Specification

## Purpose
TBD - created by archiving change app-ble-scan-telemetry-optimization. Update Purpose after archive.
## Requirements
### Requirement: Prevent BLE Scanning During Active Connection
The BLE provider and service layers SHALL prevent starting or continuing BLE advertising discovery scans while a BLE device connection is active.

#### Scenario: Connecting to a BLE device
- **WHEN** a BLE connection is initiated and established with a target device
- **THEN** any active `BleProvider` scan loop and underlying BLE discovery scan is immediately stopped and cancelled

#### Scenario: Scan request while connected
- **WHEN** `startScan()` is called while a BLE device is already connected
- **THEN** the scan request is safely ignored or returns an empty stream without starting physical radio scanning

### Requirement: Flexible BLE advertising name discovery
The BLE service SHALL accept discovered peripherals with or without the standard `RK_` name prefix.

#### Scenario: Peripheral discovered without prefix
- **WHEN** a RadioKit device advertises with a custom name lacking the `RK_` prefix
- **THEN** the BLE service discovers the device and displays it in the pairing list

