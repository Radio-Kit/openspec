# Proposal: Stop Bundling the Arduino Library into the RadioKit App

## Why

The `rk-arduino` Arduino library was split into its own repository (`Radio-Kit/RK-Arduino`) in the `split-multirepo-setup` change. However, the RadioKit Flutter app still expects a bundled `assets/rk-arduino.zip`:

1. **All 5 multiplatform release workflows fail** — they run a "Build library zip" step that zips `rk-arduino/` from the repo root, but that directory no longer exists in the RadioKit repo. Every tag push runs all five platforms and every one fails before `flutter build` is ever reached.
2. The app ships a static copy of the Arduino library that can drift out of date from the actual library repository.

The library is only consumed by AI agents / test scripts via the local REST API (`/api/library/version`, `/api/library/download`), which is used to grab library source when writing Arduino sketches. Agents can just be pointed at the canonical repository URL instead. No backward compatibility is needed — no external consumers depend on the internal `/api/library/*` endpoints.

## What Changes

- **Delete the bundling machinery**:
  - Remove the "Build library zip" step from all 5 release workflows (`release-android.yml`, `release-ios.yml`, `release-macos.yml`, `release-linux-flatpak.yml`, `release-windows.yml`).
  - Delete `scripts/build-library-zip.sh`.
  - Remove `assets/rk-arduino.zip` from `radiokit-app/pubspec.yaml` asset list.
  - Remove the `radiokit-app/assets/rk-arduino.zip` ignore entry from `.gitignore`.
- **Remove the library service and API surface**:
  - Delete `radiokit-app/lib/services/library_service.dart`.
  - Remove `LibraryService` import, field, constructor parameter, route registration, route exemption, and handlers from `remote_access_service.dart`.
  - Remove the library init/wiring and import from `remote_access_provider.dart`.
  - Remove the `libraryService` constructor parameter from `remote_access_service_stub.dart`.
  - Remove the `/api/library/version` and `/api/library/download` entries from `docs_service.dart`.
- **Point docs at the canonical repo URL**:
  - Replace the `/api/library/*` API sections in `llm-docs/API.md` (and its duplicate `radiokit-app/assets/skills/llms.txt`) with a pointer to `https://github.com/Radio-Kit/RK-Arduino`.
  - Replace the library API table rows in `SKILLS/radiokit-remote/SKILL.md` (and its duplicate `radiokit-app/assets/skills/radiokit-remote.md`) with the same URL.

## Capabilities

### New Capabilities
<!-- None -->

### Modified Capabilities
- `radiokit-release-workflows`: multiplatform release workflows no longer build/bundle the Arduino library zip.
- `radiokit-remote-access-api`: the local REST API no longer exposes library version/download endpoints.

## Impact

- **Release builds**: All 5 workflows stop failing; tag pushes produce working artifacts.
- **App bundle**: Smaller app; `rk-arduino.zip` no longer embedded as a Flutter asset.
- **Agents**: Library consumers get the source from the `Radio-Kit/RK-Arduino` repository instead of a bundled zip.
- **No backward compatibility**: `/api/library/version` and `/api/library/download` are removed without deprecation.