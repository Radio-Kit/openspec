## 1. Release Workflows

- [x] 1.1 Remove the "Build library zip" step from `release-android.yml`
- [x] 1.2 Remove the "Build library zip" step from `release-ios.yml`
- [x] 1.3 Remove the "Build library zip" step from `release-macos.yml`
- [x] 1.4 Remove the "Build library zip" step (inline `Compress-Archive`) from `release-windows.yml`
- [x] 1.5 Remove the `bash scripts/build-library-zip.sh` command from `flatpak/flatpak-flutter.yml` (the flatpak workflow itself had no bundling step; the failure came from the manifest build-commands)

## 2. Bundling Assets & Scripts

- [x] 2.1 Delete `scripts/build-library-zip.sh`
- [x] 2.2 Remove `assets/rk-arduino.zip` from `radiokit-app/pubspec.yaml` asset list
- [x] 2.3 Remove `radiokit-app/assets/rk-arduino.zip` ignore entry from `.gitignore`

## 3. Library Service & API Surface

- [x] 3.1 Delete `radiokit-app/lib/services/library_service.dart`
- [x] 3.2 Remove `LibraryService` import, init, and wiring from `remote_access_provider.dart`
- [x] 3.3 Remove `LibraryService` import, `_libraryService` field, ctor param, route registration, route exemption, and both handlers from `remote_access_service.dart`
- [x] 3.4 Remove the `libraryService` ctor param from `remote_access_service_stub.dart`
- [x] 3.5 Remove `/api/library/version` and `/api/library/download` entries from `docs_service.dart`

## 4. Docs & Skills point at the repo URL

- [x] 4.1 Update `llm-docs/API.md`: replace `/api/library/*` sections with `https://github.com/Radio-Kit/RK-Arduino`
- [x] 4.2 Update `radiokit-app/assets/skills/llms.txt` to match (duplicate of API.md)
- [x] 4.3 Update `SKILLS/radiokit-remote/SKILL.md`: replace library API table rows with the repo URL
- [x] 4.4 Update `radiokit-app/assets/skills/radiokit-remote.md` (condensed variant of SKILL.md): replace library rows with the repo URL

## 5. Verification

- [x] 5.1 Grep for residual `LibraryService`, `rk-arduino.zip`, `/api/library`, `build-library-zip` references and confirm none remain
- [x] 5.2 Run `flutter analyze` in `radiokit-app` (only pre-existing issues in untouched files; no new issues)
- [x] 5.3 Run `flutter test` in `radiokit-app`
- [x] 5.4 Confirm `llm-docs/API.md` and `radiokit-app/assets/skills/llms.txt` sections are still identical after edits
- [x] 5.5 Confirm no `/api/library` references remain and both `SKILLS/radiokit-remote/SKILL.md` and `radiokit-app/assets/skills/radiokit-remote.md` point at the repo URL (note: these two are structurally different by design — condensed vs full — pre-existing, not drift)