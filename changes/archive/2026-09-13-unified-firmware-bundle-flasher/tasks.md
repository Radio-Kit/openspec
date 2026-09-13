## 1. Firmware Bundle Parser Service

- [x] 1.1 Implement `FirmwareBundleParser` and model classes in `radiokit-app/lib/services/firmware_bundle_parser.dart` to parse ESP Web Tools `manifest.json` and extract zip parts in-memory
- [x] 1.2 Add chip family compatibility validator matching `detectedChip.family` to `manifest.chipFamily`
- [x] 1.3 Add unit tests for `FirmwareBundleParser` verifying valid zip parsing, missing manifest rejection, and multi-part offset ordering

## 2. Flasher Provider & USB Lifecycle Updates

- [x] 2.1 Update `FlasherProvider` to strictly require `SelectedBundle` (removing raw `.bin` fields)
- [x] 2.2 Update `FlasherProvider.startFlashing()` to iterate through all manifest parts and execute `writeFlash` per partition offset
- [x] 2.3 Remove redundant `_enterBootloaderMode()` pulse in `_reprepareForFlashing()` when already connected to prevent ESP32-S3 USB dropouts
- [x] 2.4 Update `flasher_screen.dart` UI to restrict file picker to `.zip` extensions and render bundle metadata (name, version, chip, partition parts)

## 3. Remote Access Service API Updates

- [x] 3.1 Update `_handleFlasherSelectFirmware` in `remote_access_service.dart` to parse `.zip` bundles with `manifest.json` and reject raw `.bin` data
- [x] 3.2 Update `_handleFlasherStatus` to return detailed bundle and multi-part information

## 4. Unified OTA Engine Bundle Support

- [x] 4.1 Update OTA update service in `radiokit-app` to accept `.zip` bundles and automatically extract the app partition (`offset 65536` / `0x10000`)
- [x] 4.2 Support optional filesystem OTA when `littlefs.bin` is present in the bundle

## 5. Release Packaging Script

- [x] 5.1 Create / update release packaging script `scripts/package_release.py` to generate standard ESP Web Tools `manifest.json` and package `.zip` release bundles from PlatformIO build outputs

## 6. Testing & End-to-End Verification

- [x] 6.1 Run all Flutter unit and provider tests
- [x] 6.2 Test USB flashing of TrackLink v3 (ESP32-S3) via remote API `10.0.0.2:7007` with the generated `.zip` bundle
- [x] 6.3 Verify TrackLink v3 boots cleanly after flash and communicates with RadioKit companion app
