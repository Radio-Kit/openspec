# Tasks: Fix App State Copy Overhead

## Implementation

- [x] **1.1** Change `setInputValue()` to mutate `_widgetState!.inputValues` directly
- [x] **1.2** Change `_handleVarUpdate()` to mutate `_widgetState!.outputValues` directly
- [x] **1.3** Change `_handleSetInput()` to mutate `_widgetState!.inputValues` directly
- [x] **1.4** Verify `copyWithInput`/`copyWithOutput` still used for initial state construction

## Verification

- [x] **2.1** Build app
- [x] **2.2** Install app on tablet
- [x] **2.3** Run 5-minute latency stress test at 10Hz — HTTP server must stay responsive
- [x] **2.4** Check post-test latency — must be <50ms immediately after test
- [x] **2.5** Run E2E hardware test — all 7 phases pass
