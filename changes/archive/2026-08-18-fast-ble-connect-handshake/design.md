## Context

Connecting to a RadioKit-enabled ESP32 over BLE currently takes between 1.5s and 4.0s before the controller UI and telemetry widgets become active.
1. During `BleService.connect()` and `BleService.connectToDevice()`, the client discovers 5 characteristics (`widget`, `fs`, `ota`, `settings`, `print`) and subscribes to each with an individual `await UniversalBle.subscribeNotifications(...)`. This produces 5 sequential round-trips over the BLE connection.
2. In `DeviceProvider._requestConfig()`, the client waits up to `kPushWaitTimeout` (configured to 3 seconds) for the firmware's proactive `CONF_DATA` push. If this push is dropped or delayed during subscription, the app stays idle for 3 seconds before sending `GET_CONF` (0x01).

## Goals / Non-Goals

**Goals:**
- Parallelize characteristic notification subscriptions in `BleService` using `Future.wait`.
- Reduce `kPushWaitTimeout` from 3 seconds to 500ms for rapid fallback to `GET_CONF`.
- Preserve idempotency and clean handling in `DeviceProvider` when `CONF_DATA` or `VAR_DATA` arrives via push, pull, or duplicate responses.
- Ensure compatibility across Android, iOS, and desktop platforms.

**Non-Goals:**
- Altering the wire protocol packet format or command IDs (0x01 `GET_CONF`, 0x02 `CONF_DATA`, 0x04 `VAR_DATA` remain unchanged).
- Modifying the firmware subscribe push implementation in `RadioKitBLE.cpp`.

## Decisions

### 1. Concurrent `Future.wait` for Characteristic Subscriptions
- **Decision**: Group all `UniversalBle.subscribeNotifications` calls into a single `Future.wait([...])` list in both `connect()` and `connectToDevice()`.
- **Rationale**: Android's `universal_ble` plugin already manages an internal GATT command executor to serialize low-level descriptor writes, while iOS/macOS natively supports concurrent notification requests. Using `Future.wait` eliminates Dart async event-loop latency between sequential awaits without risking descriptor write corruption.
- **Alternatives Considered**:
  - *Keep sequential subscriptions*: Adds 300ms–600ms of unnecessary overhead.
  - *Lazy-subscribing non-essential characteristics (FS, OTA)*: Complicates state management and requires tracking subscription state per feature.

### 2. Tuning `kPushWaitTimeout` to 500ms
- **Decision**: Update `kPushWaitTimeout` in `protocol.dart` from `Duration(seconds: 3)` to `Duration(milliseconds: 500)`.
- **Rationale**: If the ESP32 pushes `CONF_DATA` on widget subscribe, the notification arrives within 50–150ms over a local BLE connection. Waiting 500ms allows sufficient buffer for radio scheduling while preventing a 3-second freeze if the push was missed.
- **Alternatives Considered**:
  - *Always send GET_CONF immediately without waiting*: Redundant packet if push is already in flight. A 500ms window gives push the priority while keeping worst-case fallback fast.

## Risks / Trade-offs

- **[Risk] BLE Descriptor write contention on older Android devices** → `universal_ble` 2.x serializes GATT operations internally in Java/Kotlin.
- **[Risk] Duplicate CONF_DATA arrival if push arrives right as GET_CONF is sent** → `DeviceProvider._handleConfData()` is completely idempotent; reprocessing `CONF_DATA` simply re-verifies the widget list and updates state cleanly.
