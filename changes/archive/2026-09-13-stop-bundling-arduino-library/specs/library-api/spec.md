## REMOVED Requirements

### Requirement: Library version endpoint
**Reason**: The Arduino library is no longer bundled inside the app, so there is no embedded `library.json` from which the app can report a version. Agents can read version information directly from the `Radio-Kit/RK-Arduino` repository.
**Migration**: Agents consume the library from `https://github.com/Radio-Kit/RK-Arduino` instead of `GET /api/library/version`.

### Requirement: Library download endpoint
**Reason**: The Arduino library is no longer bundled as an app asset, so `GET /api/library/download` has nothing to serve.
**Migration**: Agents obtain the library source from `https://github.com/Radio-Kit/RK-Arduino` (browse, clone, or fetch the repository directly).

### Requirement: Build script creates library ZIP
**Reason**: The `rk-arduino/` directory was moved out of the RadioKit repository into `Radio-Kit/RK-Arduino`; there is no source tree to zip inside this repo, and the failing bundling step blocks all release workflows.
**Migration**: No replacement build script. The multiplatform release workflows build the Flutter app directly without generating an Arduino library asset.

### Requirement: LibraryService initialization
**Reason**: `LibraryService` exists solely to extract the bundled zip and serve the removed endpoints; with the bundle gone the service is dead code.
**Migration**: Remove `LibraryService` and its initialization from the remote access startup path.