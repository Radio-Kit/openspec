## ADDED Requirements

### Requirement: Variable Update ACK Suppression
The `RadioKitClass::_handleVarUpdate` method SHALL NOT transmit automatic `RK_CMD_ACK` packets in response to incoming `RK_CMD_VAR_UPDATE` commands. Variable state updates SHALL rely on shadow state synchronization and fast write-without-response delivery.

#### Scenario: Continuous input streaming
- **WHEN** the client streams continuous variable update packets (`RK_CMD_VAR_UPDATE`) for sliders or control inputs
- **THEN** the firmware updates widget inputs and shadow buffers immediately without emitting reverse ACK notifications over BLE or other transports
