## 1. Boot Print Flush (Library)

- [x] 1.1 Add `_bootFlushDone` bool flag to `RadioKitClass.h` private section
- [x] 1.2 Initialize `_bootFlushDone(false)` in `RadioKitClass` constructor
- [x] 1.3 Add one-shot boot flush logic in `_flushPrintBuffer()` before `isConnected()` gate
- [x] 1.4 Verify: PlatformIO BasicSwitch builds clean

## 2. Codegen Init Order (App)

- [x] 2.1 Move `enableFS()` before `startBLE()` in `json_arduino_generator.dart`
- [x] 2.2 Update comment explaining the init order rationale
- [x] 2.3 Verify: `dart analyze` passes
- [x] 2.4 Verify: 38 codegen tests pass
- [x] 2.5 Verify: 400 full Flutter tests pass
