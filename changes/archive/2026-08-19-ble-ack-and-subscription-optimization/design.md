## Context

Continuous control interactions over BLE generate high-frequency input packets (~40 Hz for slider/gas pedal drags). In the current firmware implementation, `RadioKitClass::_handleVarUpdate` responds to every packet with `rk_buildAck` over BLE notify. This floods the NimBLE notification queue, conflicts with telemetry transmissions, and causes main loop stalls. Furthermore, during BLE connection setup, `ble_service_impl.dart` issues parallel `subscribeNotifications` calls via `Future.wait`, triggering Android GATT status 17 (`GATT_BUSY`) errors.

## Goals / Non-Goals

**Goals:**
- Eliminate reverse ACK packet generation in `RadioKitClass::_handleVarUpdate`.
- Switch BLE characteristic notification subscription from parallel `Future.wait` to sequential `await` in both single-device and multi-device connection paths.
- Ensure zero `GATT_BUSY` errors on Android during connection.
- Keep the BLE outbound notification channel free for telemetry and status frames.

**Non-Goals:**
- Removing bulk protocol ACKs (e.g. FS and OTA frames retain their dedicated ACKs).
- Modifying payload formats or packet headers.

## Decisions

### Decision 1: Remove Automatic ACKs for Variable Updates
In `RadioKit.cpp`, remove `rk_buildAck` and `_sendPacket(pkt)` from `_handleVarUpdate`. Streaming inputs are continuously refreshed (latest-value semantics), making discrete packet ACKs unnecessary and counter-productive.

### Decision 2: Sequential Notification Subscription in App
In `ble_service_impl.dart`:
- Replace `Future.wait([UniversalBle.subscribeNotifications(...)])` with sequential `for` loops using `await`.
- Apply this to both `_connectDevice` (single device) and `_subscribeCharacteristicsMulti` (multi-device).

## Risks / Trade-offs

- **[Risk] App expecting ACK confirmation on discrete toggle buttons** → *Mitigation*: The app already utilizes shadow-state synchronization (`kCmdVarData`) and writes without response; removing reverse ACKs prevents queue buildup without impacting toggle responsiveness.
