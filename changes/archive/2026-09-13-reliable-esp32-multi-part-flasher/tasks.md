## 1. Flutter Dependencies Upgrade

- [x] 1.1 Update `pubspec.yaml` dependencies in `radiokit-app`
- [x] 1.2 Run `flutter pub upgrade` and ensure clean dependency resolution

## 2. Android USB Serial & Transport Synchronization

- [x] 2.1 Update `FlserialPortAdapter.write()` to await MethodChannel USB bulk transfers on Android
- [x] 2.2 Verify buffer management and error handling during baud rate transitions

## 3. Full Chip Erase & Multi-Part Flashing Pipeline

- [x] 3.1 Implement explicit full chip erase (`_flashService.eraseFlash()`) in `FlasherProvider.startFlashing()` when `eraseAll` is enabled
- [x] 3.2 Implement sequential multi-part partition writes targeting manifest offsets
- [x] 3.3 Implement aggregate multi-part progress tracking and status messaging

## 4. Verification & Testing

- [x] 4.1 Run `flutter analyze` and existing test suites to ensure zero regressions
- [x] 4.2 Verify bundle parsing and fallback mechanisms with simulated multi-part payloads
