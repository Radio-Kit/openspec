## Context

RadioKit companion app allows updating device firmware over OTA (via protocol `0xBB`). Devices can report an OTA URL (`#define RK_OTA_URL "https://github.com/..."` sent via `0x0F` / `0x8F`). However, currently the app only supports selecting a local file from storage, and `RadioKit.config.version` is not transmitted in the settings protocol.

This design introduces an integrated firmware release check mechanism in the companion app's Firmware Tab that connects to GitHub Releases, compares versions, displays release changelogs, matches board binary assets, and downloads them for direct in-app OTA flashing.

## Goals / Non-Goals

**Goals:**
- Transmit `RadioKit.config.version` from MCU to App via the `SETTINGS_RESP_DEVICE_INFO_DATA` (0x88) frame.
- Add `FirmwareReleaseService` in the companion app to query GitHub Releases API (`/releases/latest`), extract `.bin` assets, and stream binary bytes into memory.
- Implement intelligent asset matching based on device name / board identifier with user override via dropdown selector.
- Provide a clean Firmware Tab UI with auto-check on open, manual refresh, expandable release notes, and one-tap download & flash.
- End-to-end testing with real hardware (Mikro board) using test firmware releases in `https://github.com/Radio-Kit/demo-fs-assets`.

**Non-Goals:**
- Automatic silent background firmware updates (updates remain strictly user-initiated).
- Non-GitHub release repository hosting (standardizes on GitHub Releases API for open-source repositories).

## Decisions

### Decision 1: Version Transport in `DEVICE_INFO` Frame
- **Rationale**: The app already sends `SETTINGS_CMD_GET_DEVICE_INFO` (0x08) upon initial connection. Appending `[VER_LEN(1)][VERSION(N)]` to `SETTINGS_RESP_DEVICE_INFO_DATA` (0x88) avoids creating a new round-trip command while making firmware version immediately accessible.
- **Alternatives Considered**:
  - *Dedicated `GET_VERSION` command (0x10)*: Adds extra latency and protocol complexity during connection handshake.

### Decision 2: Standalone `FirmwareReleaseService`
- **Structure**:
  - `fetchLatestRelease(String otaUrl)` -> `Future<FirmwareRelease?>`
  - `downloadAsset(String downloadUrl, {Function(int, int)? onProgress})` -> `Future<Uint8List>`
- **Models**:
  - `FirmwareRelease`: `tagName`, `version`, `title`, `publishedAt`, `changelog`, `assets`
  - `ReleaseAsset`: `name`, `size`, `downloadUrl`

### Decision 3: Version Parsing and Comparison
- Strip leading `v` from tag (e.g. `v1.2.0` -> `1.2.0`).
- Compare semver components `[major, minor, patch]`. If remote > local, trigger `updateAvailable`.

### Decision 4: Asset Matching Strategy
- Filter assets for `.bin` extension.
- Priority:
  1. Exact or case-insensitive substring match of device name (e.g. `MIKRO_V2` matches `MIKRO_V2.bin`).
  2. Single binary asset if only one exists (e.g. `firmware.bin`).
  3. First binary asset with user dropdown selector to change selection.

### Decision 5: Test Firmware and Hardware E2E Verification
- Target: Mikro board connected via USB/Serial or BLE.
- Repository: `https://github.com/Radio-Kit/demo-fs-assets`.
- Build tag define: Version configured via PlatformIO build flags (e.g. `-D RK_VERSION=\"2.1.0\"` or `RadioKit.config.version`).

## Risks / Trade-offs

- **[Risk] GitHub API Rate Limiting for unauthenticated requests (60 req/hr per IP)**:
  - *Mitigation*: Releases API is only queried on opening the tab or on manual refresh button click; results can be cached during the active session.
- **[Risk] Large binary size exceeding device RAM on mobile device**:
  - *Mitigation*: ESP32 firmware binaries are typically 1-2 MB, which easily fits in app RAM (`Uint8List`) before streaming chunks to MCU via OTA.
