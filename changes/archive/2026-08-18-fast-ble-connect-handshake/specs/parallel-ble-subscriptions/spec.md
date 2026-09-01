## ADDED Requirements

### Requirement: Concurrent BLE characteristic subscriptions
When connecting to a RadioKit BLE device, the app SHALL initiate and await notification subscriptions on all discovered characteristics concurrently using `Future.wait` rather than sequentially.

#### Scenario: Device with all standard characteristics discovered
- **WHEN** a BLE connection is established and characteristics (widget, fs, ota, settings, print) are discovered
- **THEN** notification subscriptions for all discovered characteristics are dispatched concurrently in parallel, completing as soon as all subscriptions resolve without sequential event-loop delays between each characteristic

#### Scenario: Multi-device connection dispatch
- **WHEN** a BLE connection is initiated via `connectToDevice()` in multi-device mode
- **THEN** notification subscriptions for discovered characteristics on that device are also dispatched concurrently via `Future.wait`
