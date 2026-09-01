# app-telemetry-output-sync Specification

## Purpose
TBD - created by archiving change app-ble-scan-telemetry-optimization. Update Purpose after archive.
## Requirements
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

### Requirement: Telemetry Widget ID Preservation in Designer JSON
The `widgetConfigsToDesignerJson` function SHALL serialize each telemetry widget in the top-level `telemetry` list with an integer `id` matching its `widgetId`, in addition to `label` and optional `icon`.

#### Scenario: Reconstructing designer config from wire widget configs
- **WHEN** `widgetConfigsToDesignerJson` processes a list of `WidgetConfig` objects containing `kWidgetTelemetry` widgets
- **THEN** the returned map contains a `telemetry` list where each element includes `id`, `label`, and optional `icon`

### Requirement: Active Link Telemetry Value Resolution
The active link telemetry view on the Models tab SHALL resolve telemetry items directly from the device's telemetry definitions and look up their live string values using the corresponding `widgetId` in `dp.telemetryValues` across both single-page and multi-page configurations.

#### Scenario: Displaying live telemetry on active model card
- **WHEN** the connected device receives `VAR_UPDATE` packets for telemetry widgets
- **THEN** the active link card displays the live string values (such as battery percentage and speed) formatted with their respective units, updating dynamically from `dp.telemetryValues`

### Requirement: Firmware Page-Agnostic Telemetry Inclusion
The firmware `RadioKitClass` serialization routines (`_buildConfPayload`, `_buildVarPayload`, and `_buildMetaPayload`) SHALL include `RK_TYPE_TELEMETRY` widgets regardless of whether the active page index matches the widget's default page index.

#### Scenario: Telemetry payload generation on secondary page
- **WHEN** the vehicle controller operates on a page other than page 0 (e.g. Locomotive on page 1)
- **THEN** `_buildConfPayload` and `_buildVarPayload` include all registered `RK_Telemetry` widgets in the wire payloads

