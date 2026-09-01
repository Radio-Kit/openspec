# Fix App State Copy Overhead

## Problem

After 5 minutes of 10Hz command traffic (3000 commands), the Flutter app's HTTP server freezes. The app's main isolate event loop becomes saturated, causing:

1. **HTTP server blocks** — PUT requests to `/api/widgets/0` timeout
2. **Touch UI becomes unresponsive** — gesture events queue behind rebuild work
3. **App appears frozen** — must force-restart to recover

## Root Cause

Every `copyWithInput()` and `copyWithOutput()` call creates a **new Map** and a **new RadioWidgetState** object:

```dart
RadioWidgetState copyWithInput(int widgetId, List<int> values) {
    final newInputs = Map<int, List<int>>.from(inputValues);  // ← NEW MAP!
    newInputs[widgetId] = values;
    return RadioWidgetState(inputValues: newInputs, outputValues: outputValues);
}
```

Over 5 minutes at 10Hz:
- 3000 `copyWithInput()` calls → 3000 new Maps + 3000 new RadioWidgetState objects
- 3000 `copyWithOutput()` calls → 3000 new Maps + 3000 new RadioWidgetState objects
- **Total: 12,000+ heap allocations in 5 minutes**

This causes:
- GC pressure that pauses the event loop
- Memory fragmentation
- Event loop saturation from excessive object creation

## Solution

Mutate the existing state in-place instead of creating copies. Use `_widgetState!` directly:

```dart
// Before: creates new object every time
_widgetState = _widgetState?.copyWithInput(widgetId, values);

// After: mutates existing object in-place
_widgetState!.inputValues[widgetId] = values;
notifyListeners();
```

### Key Changes

1. **`setInputValue()`** — mutate `inputValues` map directly instead of `copyWithInput()`
2. **`_handleVarUpdate()`** — mutate `outputValues` directly instead of `copyWithOutput()`
3. **`_handleSetInput()`** — mutate `inputValues` directly instead of `copyWithInput()`
4. **Keep `copyWithInput`/`copyWithOutput`** — still needed for initial state construction

### Why This Is Safe

- `notifyListeners()` still triggers Flutter rebuilds correctly
- The state object is the same reference — Flutter's `==` comparison works via `notifyListeners()`
- No functional change — same behavior, zero heap allocations per update

## Scope

- **RadioKit app**: `device_provider.dart` — mutate state in-place
- **RadioKit app**: `widget_config.dart` — keep copyWith methods for construction only
- **Firmware**: No changes needed

## Verification

1. 5-minute latency stress test at 10Hz — HTTP server must remain responsive
2. Post-test latency check — must be <50ms immediately after test
3. Touch UI responsiveness — must not freeze during or after sustained use
4. E2E hardware test — all 7 phases pass
