## ADDED Requirements

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
