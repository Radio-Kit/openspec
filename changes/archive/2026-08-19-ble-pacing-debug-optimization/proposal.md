## Why

When high-frequency control inputs (such as continuous slider drags or touch sweeps) occur over BLE, the ESP32 main control loop experiences severe stalls. This is caused by:
1. `RadioKitBLE::sendPacket` employing a synchronous blocking retry loop with progressive backoff (`delay(10)` to `delay(100)`, totaling up to 550ms) whenever GATT characteristic notifications fail due to stack congestion.
2. Verbose logging (`CORE_DEBUG_LEVEL=5`) emitting high-frequency NimBLE GATT write/status logs on every touch event, which saturates the serial bus and introduces CPU interrupt overhead.

Fixing these issues ensures the MCU main loop runs smoothly at >100Hz without stuttering motor control or telemetry freezes.

## What Changes

- **Non-Blocking / Differentiated BLE Packet Sending**: Differentiate loss-tolerant real-time packets (widget telemetry `0x55`, debug stream `0xDD`) from critical/bulk packets (`0xAA` FS, `0xBB` OTA, `0xCC` Settings). Loss-tolerant packets drop immediately or retry at most once with `taskYIELD()` without blocking delays. Critical/bulk packets use bounded, low-latency pacing.
- **Serial Debug Level Reduction**: Lower `CORE_DEBUG_LEVEL` in platform configurations to eliminate NimBLE GATT debug chatter while preserving application-level structured telemetry (`Serial.printf`).

## Capabilities

### New Capabilities
- `ble-send-pacing`: Differentiated non-blocking transmission and congestion handling for BLE packets in `RadioKitBLE`.

### Modified Capabilities
<!-- None -->

## Impact

- `rk-arduino`: `src/connection/RadioKitBLE.cpp` packet dispatch and retry loop.
- Platform configurations / projects (`platformio.ini`): Reduced debug log levels.
- Real-time performance: Microcontroller main loop cycle latency drops from >500ms under congestion to <1ms.
