## Context

The repository currently contains independent workflows for Android, Windows, macOS, and Linux Flatpak in `.github/workflows/`. However, they currently suffer from several issues:
1. They attempt to run `gh release create` directly, leading to race condition failures when multiple jobs complete at similar times for the same git tag.
2. Windows generates only raw zip files without an installer executable.
3. macOS produces an uncompressed or raw zip instead of a user-friendly `.dmg`.
4. Artifact naming conventions are inconsistent across platforms.

## Goals / Non-Goals

**Goals:**
- Maintain 4 dedicated platform workflow files (`release-android.yml`, `release-windows.yml`, `release-macos.yml`, `release-linux-flatpak.yml`).
- Use Inno Setup on `windows-latest` to build a `.exe` installer + `.zip` archive.
- Use `create-dmg` / `hdiutil` on `macos-latest` (Apple Silicon ARM64) to package `RadioKit.app` into `.dmg`.
- Build and package Linux Flatpak bundle (`.flatpak`).
- Build and package Android APK (`.apk`).
- Use `softprops/action-gh-release@v2` for reliable, race-free soft release uploads.
- Standardize artifact naming to `radiokit-<version>-<platform>.<ext>`.

**Non-Goals:**
- Signing macOS binaries with paid Apple Developer Certificates or notarization (can be added later when certificates are configured).
- Microsoft Authenticode code signing with paid EV certificates.
- Google Play Store automated uploading (scope is GitHub Releases).

## Decisions

### 1. Inno Setup for Windows Installer
- **Decision**: Create an Inno Setup script (`windows/installer/RadioKit.iss` or generated dynamically in CI) and run `iscc`.
- **Rationale**: Inno Setup is lightweight, pre-installed on GitHub Actions `windows-latest`, and creates clean modern installers with start menu entries, desktop shortcuts, and uninstaller.
- **Alternatives considered**: NSIS (more complex scripting) or MSIX (requires code signing certificates).

### 2. Apple Silicon DMG via create-dmg / hdiutil
- **Decision**: Use `create-dmg` (or native `hdiutil create`) on `macos-latest` runner.
- **Rationale**: `macos-latest` runs Apple Silicon (M1/M2/M3). Creating a DMG provides a standard macOS installation experience with an `/Applications` symlink.

### 3. Softprops Action for Soft GitHub Releases
- **Decision**: Replace manual `gh release create || gh release upload` scripts with `softprops/action-gh-release@v2`.
- **Rationale**: `softprops/action-gh-release` handles tag creation, draft states, asset clobbering, and exponential backoff retries when multiple jobs upload assets to the same release concurrently.

### 4. Standardized Output Naming Convention
- **Naming Pattern**:
  - Windows Installer: `radiokit-${VERSION}-windows-x64-setup.exe`
  - Windows Portable: `radiokit-${VERSION}-windows-x64.zip`
  - macOS DMG: `radiokit-${VERSION}-macos-arm64.dmg`
  - Linux Flatpak: `radiokit-${VERSION}-linux-x86_64.flatpak`
  - Android APK: `radiokit-${VERSION}-android.apk`

## Risks / Trade-offs

- **[Risk]** Unsigned macOS DMG triggers Gatekeeper warning on modern macOS versions.
  - *Mitigation*: Provide clear documentation in release notes on how to run via `Right Click -> Open` or `xattr -cr /Applications/RadioKit.app`.
- **[Risk]** Long build times for Flatpak (4-6 min) compared to Android/Windows.
  - *Mitigation*: Workflows run fully independently in parallel, so Flatpak build time does not delay availability of other platform assets.
