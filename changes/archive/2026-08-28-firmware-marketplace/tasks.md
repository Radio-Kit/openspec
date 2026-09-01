## 1. Marketplace Service & Binary Parsing

- [x] 1.1 Implement `FirmwareMarketplaceService` in `radiokit-app/lib/services/firmware_marketplace_service.dart` to manage saved repositories in `SharedPreferences` with default curated community sources (`DragonRailway/RC_Engine`, `Radio-Kit/demo-fs-assets`).
- [x] 1.2 Implement standardized binary filename parser (`<Project>-<Version>-<Chip>[-<BoardOrVariant>][-<Type>].bin`) and chip/type matcher in `FirmwareMarketplaceService`.
- [x] 1.3 Add unit tests for `FirmwareMarketplaceService` (repo persistence, filename parsing, chip matching) in `radiokit-app/test/services/firmware_marketplace_service_test.dart`.

## 2. QR Code Scanner & Add Repo Dialog

- [x] 2.1 Implement `QrRepoScannerModal` in `radiokit-app/lib/screens/home/qr_repo_scanner_modal.dart` supporting camera QR scanning on mobile and clipboard paste on desktop.
- [x] 2.2 Add QR payload parser supporting plain GitHub URLs (`https://github.com/owner/repo`) and deep links (`radiokit://firmware?url=...`).

## 3. Flasher Tab Marketplace UI

- [x] 3.1 Replace `_MarketplacePlaceholder` in `radiokit-app/lib/screens/home/flasher_tab.dart` with an active `_MarketplaceSection` containing saved repository list and add/scan action buttons.
- [x] 3.2 Build the Repository Release Card with repo metadata, release tag badge, expandable changelog viewer, and binary asset list with chip compatibility indicators.
- [x] 3.3 Wire the "DOWNLOAD & FLASH" action to download binary bytes directly into memory and trigger `FlasherProvider.flashCustomFirmware()`.

## 4. Verification & Testing

- [x] 4.1 Run Flutter unit test suite (`flutter test`) to verify all unit tests pass.
- [x] 4.2 Verify real-world discovery with `DragonRailway/RC_Engine` and `Radio-Kit/demo-fs-assets` repositories.
