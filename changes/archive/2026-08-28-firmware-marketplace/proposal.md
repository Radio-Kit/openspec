## Why

Currently, flashing custom firmware onto ESP32 devices via the companion app's Flasher Tab requires manually selecting a local `.bin` file from the filesystem. Users and makers lack an integrated way to discover, browse, and directly flash pre-built binaries from community GitHub repositories (such as DragonRailway or RadioKit starter kits), or quickly import projects by scanning QR codes printed on project READMEs, kits, and device boxes.

## What Changes

- **Firmware Marketplace in Flasher Tab**: Replaces the static `_MarketplacePlaceholder` with an active, decentralized catalog browser that lists community GitHub repositories, fetches latest release metadata, and exposes release assets.
- **Repository Sources & Persistence**: Adds local persistence for saved repository sources in `SharedPreferences`, pre-populated with curated community repositories (`DragonRailway/RC_Engine`, `Radio-Kit/demo-fs-assets`).
- **QR Code Scanner**: Implements an in-app QR scanner modal (and desktop clipboard/paste fallback) supporting standard GitHub URLs and deep links (`radiokit://firmware?url=...`).
- **Standardized Binary Naming & Chip Matching**: Standardizes community binary names (`<Project>-<Version>-<Chip>[-<BoardOrVariant>][-<Type>].bin`), parses chips (`esp32`, `esp32s3`, `esp32c3`, `esp32c6`, `esp32s2`), filters flash types (`factory` vs `ota`), and auto-selects/badges binaries matching the connected USB ESP32 hardware.
- **One-Click Stream & Flash**: Downloads selected `.bin` asset bytes over HTTP directly into memory and triggers the `FlasherProvider` serial bootloader flashing pipeline.

## Capabilities

### New Capabilities
- `firmware-marketplace`: Decentralized GitHub repository firmware catalog, persistent saved sources management, QR code scanner, standardized binary parsing and board matching, and one-click USB bootloader flashing in the Flasher Tab.

### Modified Capabilities
<!-- None -->

## Impact

- **App Screens**: `radiokit-app/lib/screens/home/flasher_tab.dart` (replaces placeholder with marketplace hub, repo list, add dialog, and release cards).
- **App Services**: New `FirmwareMarketplaceService` (for managing saved repos, parsing binary filenames, and querying release assets) or extensions to `FirmwareReleaseService`.
- **Dependencies**: Adds QR code scanning support for mobile platforms (e.g. `mobile_scanner` or platform-appropriate QR package).
- **Firmware / Hardware**: Zero firmware changes required; leverages the existing ESP32 ROM bootloader protocol and `FlasherProvider`.
