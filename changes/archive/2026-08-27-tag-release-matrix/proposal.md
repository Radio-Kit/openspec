## Why

Currently, release workflows are isolated, suffer from race conditions when concurrently creating GitHub releases on a tag push, and produce conflicting or unstandardized artifact names without platform-native installer packaging (such as Windows Inno Setup `.exe` or macOS `.dmg`). We need automated, standardized builds triggered on tag pushes (`v*.*.*`) and manual workflow dispatches that package and attach Windows installer, macOS Apple Silicon DMG, Linux Flatpak bundle, and Android APK binaries to GitHub Releases safely.

## What Changes

- Add Windows Inno Setup installer packaging (`.exe`) alongside portable archive builds in `.github/workflows/release-windows.yml`.
- Add macOS Apple Silicon (`macOS ARM64`) Disk Image packaging (`.dmg`) in `.github/workflows/release-macos.yml`.
- Enhance Linux Flatpak workflow to output versioned standalone bundles (`.flatpak`) in `.github/workflows/release-linux-flatpak.yml`.
- Standardize Android release build to generate versioned release APK in `.github/workflows/release-android.yml`.
- Standardize artifact naming conventions across all platforms: `radiokit-v<version>-<os>-<arch>.<ext>`.
- Use soft release publishing (`softprops/action-gh-release@v2`) across all 4 release workflows to eliminate race conditions when publishing concurrent multi-platform builds for the same release tag.
- Support both tag triggers (`v*.*.*`) and manual triggers (`workflow_dispatch`).

## Capabilities

### New Capabilities
- `multiplatform-release-matrix`: Automated CI/CD build and packaging pipeline for Windows (`.exe`), macOS ARM64 (`.dmg`), Linux (`.flatpak`), and Android (`.apk`) attached directly to GitHub Releases on tag push or manual dispatch.

### Modified Capabilities
<!-- None -->

## Impact

- `.github/workflows/release-android.yml`: Updated triggers, artifact naming, and soft release publishing.
- `.github/workflows/release-windows.yml`: Added Inno Setup script compiler, standardized output naming, and soft release publishing.
- `.github/workflows/release-macos.yml`: Added Apple Silicon DMG creation, standardized output naming, and soft release publishing.
- `.github/workflows/release-linux-flatpak.yml`: Standardized bundle naming and soft release publishing.
- Added Inno Setup configuration template for Windows installer packaging in repo scripts/assets.
