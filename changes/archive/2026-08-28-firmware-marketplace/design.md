## Context

The RadioKit companion app includes a built-in ESP32 serial bootloader flasher (`FlasherTab`) and a live device OTA firmware updater (`FirmwareTab`). Currently, `FlasherTab` contains a static `_MarketplacePlaceholder` and requires users to manually select local binary files. Makers sharing DIY robotics and IoT projects (e.g. DragonRailway locomotives, RC cars, sensor nodes) currently have to instruct users to download `.bin` files manually from GitHub and load them into the app.

This design introduces an integrated **Firmware Marketplace** in the Flasher Tab to browse releases from saved GitHub repositories, scan QR codes from project READMEs and kit packaging, match binary targets against connected ESP32 hardware, and flash directly over USB.

## Goals / Non-Goals

**Goals:**
- Replace `_MarketplacePlaceholder` in `FlasherTab` with an active repository catalog and release explorer.
- Persist a list of saved repository sources in `SharedPreferences` with default community repositories (`DragonRailway/RC_Engine`, `Radio-Kit/demo-fs-assets`).
- Support adding repositories via direct text input, clipboard paste, and camera-based QR code scanning (handling plain URLs and `radiokit://firmware?url=...` deep links).
- Standardize community firmware binary naming (`<Project>-<Version>-<Chip>[-<BoardOrVariant>][-<Type>].bin`).
- Implement intelligent chip family matching and badging (`esp32`, `esp32s3`, `esp32c3`, `esp32s2`, `esp32c6`) with automatic selection of matching binaries.
- Stream `.bin` assets directly over HTTP into `FlasherProvider` for one-click USB flashing.

**Non-Goals:**
- Centralized server / marketplace backend: Relies entirely on GitHub Releases API without intermediary servers.
- Compiling code on-device: Only downloads and flashes pre-built binary assets.
- Replacing live-device OTA flow: The marketplace in `FlasherTab` focuses on USB bootloader flashing, while `FirmwareTab` handles live device OTA updates.

## Decisions

### 1. Standardized Binary Naming Pattern
$$\texttt{<Project>-<Version>-<Chip>[-<BoardOrVariant>][-<Type>].bin}$$

- **Project**: Root project identifier (e.g. `RC_Engine`, `BasicSwitch`).
- **Version**: Semver string suffixed immediately after project (e.g. `v1.0.0`, `v2.1.0`).
- **Chip**: Hardware family token (`esp32`, `esp32s3`, `esp32c3`, `esp32s2`, `esp32c6`).
- **Board / Variant** (optional): Specific PCB hardware or feature configuration (e.g. `MIKRO_V2`, `GTRACK`, `sound`).
- **Type**: Flash mode indicator:
  - `factory` $\rightarrow$ Complete flash image starting at `0x0000` for `FlasherTab`.
  - `ota` $\rightarrow$ Application partition only starting at `0x10000` for `FirmwareTab`.
- **Fallback**: Any `.bin` file without this exact structure is still displayed and selectable with generic badges.

### 2. Architecture & Data Flow

```
┌─────────────────────────────────────────────────────────────┐
│                    Marketplace Repository                   │
│  • url: String                                              │
│  • owner: String, repo: String                              │
│  • isDefault: bool                                          │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│                FirmwareMarketplaceService                   │
│  • getSavedRepos() / addRepo(url) / removeRepo(url)         │
│  • fetchRepoReleases(url) / parseBinaryName(filename)       │
│  • matchBestBinary(binaries, connectedChip, connectedBoard) │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│                     FlasherTab Hub                          │
│  • Saved Repository Expansion Tiles                         │
│  • Release Badges & Changelog Viewer                        │
│  • Binary Selection Cards with Chip Compatibility Indicator │
│  • "DOWNLOAD & FLASH" Action → Streams into FlasherProvider │
└─────────────────────────────────────────────────────────────┘
```

### 3. QR Code Scanner & Deep Link Ingestion
- **Payloads Accepted**:
  - `https://github.com/{owner}/{repo}`
  - `github.com/{owner}/{repo}`
  - `radiokit://firmware?url={url}[&asset={assetName}]`
- **Platform Strategy**:
  - Mobile (Android/iOS): Modal with camera QR scanner (`mobile_scanner` package) + manual input.
  - Desktop/Web: Manual input modal with "Paste from Clipboard" action.

### 4. Flasher Provider Integration
- When "DOWNLOAD & FLASH" is clicked, `FirmwareReleaseService.downloadAssetBytes()` downloads the `.bin` into an in-memory `Uint8List` with a progress indicator.
- The downloaded bytes are passed directly to `flasher.setCustomFirmwareBytes(bytes, name: assetName)` and the flashing process is initiated.

## Risks / Trade-offs

- **[GitHub API Rate Limits]** → Unauthenticated GitHub API calls are limited to 60 requests/hour per IP.
  - *Mitigation*: Cache release metadata in-memory for 15 minutes; only re-fetch on explicit pull-to-refresh or app launch.
- **[Non-Standard Binary Names in 3rd Party Repos]** → Some repos may not follow the `<Project>-<Version>-<Chip>...` pattern.
  - *Mitigation*: Filename parser uses resilient regex with fuzzy substring matching; any `.bin` asset is made available for selection even if chip tokens cannot be determined.
- **[Camera Permissions on Mobile]** → Users may deny camera access for QR scanning.
  - *Mitigation*: Gracefully handle permission denials and keep manual URL paste visible at all times.
