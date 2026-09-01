## MODIFIED Requirements

### Requirement: App acquires config without a fixed startup delay

The app SHALL NOT wait a fixed sleep after transport connect before requesting or receiving config. Non-BLE transports SHALL send GET_CONF immediately after connect; BLE SHALL wait a short window (500ms) for the device push and immediately fall back to sending GET_CONF if the push does not arrive within that window.

#### Scenario: Serial/WiFi/Cloud connection
- **WHEN** the app connects via Serial, WiFi, or Cloud
- **THEN** the app sends GET_CONF immediately after connect (no fixed delay) and marks the connection connected when CONF_DATA arrives

#### Scenario: BLE connection with push-capable firmware
- **WHEN** the app connects via BLE and the device pushes CONF_DATA on subscribe
- **THEN** the app consumes the pushed CONF_DATA without sending GET_CONF, within a 500ms wait window

#### Scenario: BLE connection where the push does not arrive
- **WHEN** the 500ms BLE wait window expires without CONF_DATA
- **THEN** the app sends GET_CONF immediately and retries with the existing timeout/retry loop

#### Scenario: Config content equivalence
- **WHEN** the app receives CONF_DATA from a push versus from a GET_CONF response
- **THEN** the parsed config, connection state transition, and downstream rendering are identical (push is indistinguishable from a response)
