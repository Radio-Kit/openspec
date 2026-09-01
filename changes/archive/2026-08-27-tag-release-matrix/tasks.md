## 1. Windows Packaging & Workflow

- [x] 1.1 Create Inno Setup script template for Windows desktop build
- [x] 1.2 Update `.github/workflows/release-windows.yml` to compile Inno Setup installer `.exe` and `.zip`
- [x] 1.3 Configure `softprops/action-gh-release@v2` soft release publishing in Windows workflow

## 2. macOS Packaging & Workflow

- [x] 2.1 Add DMG packaging script / step for Apple Silicon (`macos-latest` ARM64)
- [x] 2.2 Update `.github/workflows/release-macos.yml` to generate `radiokit-<version>-macos-arm64.dmg`
- [x] 2.3 Configure `softprops/action-gh-release@v2` soft release publishing in macOS workflow

## 3. Linux Flatpak Packaging & Workflow

- [x] 3.1 Update `.github/workflows/release-linux-flatpak.yml` to produce versioned bundle `radiokit-<version>-linux-x86_64.flatpak`
- [x] 3.2 Configure `softprops/action-gh-release@v2` soft release publishing in Linux Flatpak workflow

## 4. Android Packaging & Workflow

- [x] 4.1 Update `.github/workflows/release-android.yml` to produce versioned APK `radiokit-<version>-android.apk`
- [x] 4.2 Configure `softprops/action-gh-release@v2` soft release publishing in Android workflow

## 5. Verification & Validation

- [x] 5.1 Verify YAML workflow syntax and triggers across all 4 release workflows
- [x] 5.2 Validate tag matching patterns and workflow_dispatch inputs
