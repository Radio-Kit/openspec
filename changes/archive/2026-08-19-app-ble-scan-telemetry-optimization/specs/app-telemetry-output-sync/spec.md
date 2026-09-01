## ADDED Requirements

### Requirement: Telemetry Widget Output Sizing
The protocol model SHALL define `kWidgetTelemetry` (`0x0A`) with an output size of 33 bytes in `kWidgetOutputSize` so telemetry widgets are correctly recognized as output-bearing widgets.

#### Scenario: Receiving telemetry variable update
- **WHEN** the MCU transmits a `VAR_UPDATE` frame for a telemetry widget (e.g. Battery, Speed)
- **THEN** the app classifies the widget as an output, extracts the string payload, and updates `_telemetryValues` without treating it as an input bounce

### Requirement: App-Side ACK Suppression on Incoming Updates
The app SHALL NOT transmit reverse `buildAck` packets to the device in response to incoming `VAR_UPDATE`, `SET_INPUT`, or `META_UPDATE` frames.

#### Scenario: High-frequency telemetry updates from MCU
- **WHEN** periodic telemetry `VAR_UPDATE` packets arrive from the MCU over BLE
- **THEN** the app processes the updates locally without transmitting reply ACK packets over BLE
