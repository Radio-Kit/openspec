## Context

In `RadioKitBLE.cpp`, `sendPacket` handles packet segmentation and transmission over NimBLE GATT characteristics. Under high frame throughput, `target->notify(...)` fails when the NimBLE packet queue or descriptor pool is exhausted. Currently, `sendPacket` executes up to 10 retries with increasing `delay(backoff)` (10ms, 20ms, 30ms... up to 100ms), which locks the main MCU loop for up to 550ms. Additionally, verbose core debug logging (`CORE_DEBUG_LEVEL=5`) outputs serial logs for every GATT event, adding significant latency.

## Goals / Non-Goals

**Goals:**
- Eliminate long blocking `delay()` loops in `RadioKitBLE::sendPacket`.
- Differentiate real-time / loss-tolerant frames (`0x55`, `0xDD`) from critical/bulk frames (`0xAA`, `0xBB`, `0xCC`).
- Bound the maximum latency of any `sendPacket` invocation so the MCU control loop stays responsive (>100Hz).
- Reduce core debug logging levels across build profiles to eliminate serial bus contention.

**Non-Goals:**
- Rewriting NimBLE internal buffer queue mechanisms.
- Changing the protocol frame structure or characteristic UUIDs.

## Decisions

### Decision 1: Protocol Start Byte Inspection & Differentiated Pacing
In `RadioKitBLE::sendPacket`, inspect `safeBuf[0]`:
- **Real-Time Stream (`RK_START_BYTE` 0x55, `RK_PRINT_START_BYTE` 0xDD)**: Max 1 retry. On failure, invoke `taskYIELD()` or immediate yield, and drop the frame if unable to send. This prevents stalling the vehicle control loop.
- **Reliable / Bulk Transfer (`RK_FS_START_BYTE` 0xAA, `RK_OTA_START_BYTE` 0xBB, `RK_SETTINGS_START_BYTE` 0xCC)**: Max 5 retries with `delay(1)` (1ms sleep) to allow NimBLE TX buffers to clear without stalling the main loop for more than ~5ms.

*Alternative considered*: Complete asynchronous FreeRTOS queue. Rejected because simple cooperative yield and fast drop for real-time telemetry achieves the latency goal without allocating additional heap buffers on memory-constrained MCUs.

### Decision 2: Reduction of Inter-chunk Delay
Reduce the inter-chunk delay for multi-MTU transfers from `delay(5)` to `delay(1)` or `taskYIELD()`.

### Decision 3: Lower `CORE_DEBUG_LEVEL`
Change `CORE_DEBUG_LEVEL` in platform build flags to `1` (Error) or `2` (Warn).

## Risks / Trade-offs

- **[Risk] Packet loss for telemetry under extreme BLE congestion** → *Mitigation*: Telemetry and widget state frames are continuously sent; loss of an intermediate frame is preferable to stalling motor control/PWM.
- **[Risk] Slower transfer for bulk FS if queue fills** → *Mitigation*: 5 retries with 1ms delay provide sufficient clearance for GATT TX buffers without blocking the loop.
