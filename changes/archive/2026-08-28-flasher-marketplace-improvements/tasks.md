## 1. Metadata and Service Enhancements

- [x] 1.1 Update `MarketplaceBinaryInfo` and `parseBinaryFilename` in `radiokit-app/lib/services/firmware_marketplace_service.dart` to extract board, variant, and human-readable displayName
- [x] 1.2 Update unit tests in `radiokit-app/test/services/firmware_marketplace_service_test.dart` for board and variant parsing

## 2. Flasher UI Improvements

- [x] 2.1 Update `_MarketplaceSectionState` in `radiokit-app/lib/screens/home/flasher_tab.dart` to make repository cards collapsed by default
- [x] 2.2 Filter out OTA binaries (`*.isOta`) from the USB flasher binary list in `_MarketplaceSection`
- [x] 2.3 Redesign binary asset items into human-readable cards showing Board header, Chip, Variant, Size, and asset filename
- [x] 2.4 Update action button to "SELECT FIRMWARE", which downloads the binary and stages it into `FlasherProvider` without auto-starting flash

## 3. Workflow & Packaging Update

- [x] 3.1 Update `/home/sun/Apps/RCKIT/RC_brain/scripts/package_release.py` to remove `-factory` suffix from release asset filenames

## 4. Verification and Testing

- [x] 4.1 Run flutter analyze and flutter tests to ensure all tests pass
