## 1. BLE Transport Pacing Optimization

- [x] 1.1 Update `RadioKitBLE::sendPacket` in `rk-arduino/src/connection/RadioKitBLE.cpp` to classify packets by protocol start byte (`safeBuf[0]`).
- [x] 1.2 Implement non-blocking fast yield for real-time widget/print packets (`0x55`, `0xDD`), eliminating 550ms exponential `delay()` backoff.
- [x] 1.3 Implement bounded low-latency retries (max 5 retries, 1ms delay) for critical/bulk packets (`0xAA`, `0xBB`, `0xCC`).
- [x] 1.4 Reduce inter-chunk pacing delay in multi-MTU transfers from `delay(5)` to `delay(1)` / `taskYIELD()`.

## 2. Debug Logging Level Configuration

- [x] 2.1 Update platform configurations / examples to set `CORE_DEBUG_LEVEL=1` (Error) or `2` (Warning) to eliminate high-frequency NimBLE GATT write logging.
