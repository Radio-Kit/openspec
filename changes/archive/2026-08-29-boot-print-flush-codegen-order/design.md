## Context

During BLE connection debugging on RC_brain (TRACKLINK_V3, ESP32-S3), two issues made diagnosis difficult:
1. All `RadioKit.print()` boot messages were invisible on serial
2. The codegen init order caused LittleFS to mount after BLE started, risking flash DMA corruption

## Goals / Non-Goals

**Goals:**
- Make boot-time RadioKit.print() messages visible on raw Serial
- Eliminate the LittleFS double-mount race by reordering init calls
- Zero behavioral change for connected clients

**Non-Goals:**
- Changing the connected-mode print behavior (rate limiting stays)
- Refactoring ConfigParser to use RKFs (separate RC_brain fix)

## Decisions

### Decision 1: One-shot boot flush via Serial.write()
Add `_bootFlushDone` flag. On first `_flushPrintBuffer()` call when not connected, drain the circular buffer via raw `Serial.write()` and set the flag. Subsequent calls fall through to the normal `isConnected()` gate.

Why `Serial.write()` instead of `Serial.print()`: The buffer contains raw bytes (including newlines), so `Serial.write()` is a direct passthrough without formatting overhead.

### Decision 2: enableFS() before startBLE() in codegen
Move `RadioKit.enableFS()` before `RadioKit.startBLE()` in the generated `initRadioKit()`. This mounts LittleFS while the SPI flash interface is idle, before NimBLE starts using it for advertising data.

The existing LittleFS guard in `RKFs::begin()` (fix #2) remains as a safety net for firmware that hasn't been re-flashed with the new codegen order.

## Risks
- Low: Boot flush only affects the invisible pre-connection path
- Low: Init order change is backward-compatible (old firmware still works via the LittleFS guard)
