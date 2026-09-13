## 1. Android USB Permission & Flasher Engine Fixes

- [x] 1.1 Update `RawUsbPlugin.kt` and `FlserialPlugin.kt` to register permission receivers with `RECEIVER_EXPORTED` on API 33+ with `FLAG_MUTABLE` PendingIntent
- [x] 1.2 Update `flasher_provider.dart` to set `compress: false` and `verify: true` in `FlashParameters`
- [x] 1.3 Add post-flash hardware reset sequence (DTR/RTS pulse & transport disconnect) in `flasher_provider.dart`
- [x] 1.4 Validate build of `radiokit-app` with unit / widget tests

## 2. Firmware Bundles for TrackLink V2

- [x] 2.1 Build and bundle `Filesystem_LED` firmware (bootloader, partitions, app bin for ESP32-S3 with LittleFS and LED on GPIO 42/40)
- [x] 2.2 Build and bundle `BasicSwitch` firmware (bootloader, partitions, app bin for ESP32-S3 with BasicSwitch on GPIO 42)

## 3. Deploy and End-to-End Verification on Android Tablet

- [x] 3.1 Install updated RadioKit app on Android tablet via ADB
- [x] 3.2 Flash `Filesystem_LED` via Remote API / App Flasher, verify MD5 verification and reboot
- [x] 3.3 Test LittleFS file operations and LED toggle on TrackLink V2
- [x] 3.4 Flash `BasicSwitch` via Remote API / App Flasher, verify MD5 verification and reboot
- [x] 3.5 Test BasicSwitch bidirectional sync and verify TrackLink V2 built-in LED (L0) toggling
