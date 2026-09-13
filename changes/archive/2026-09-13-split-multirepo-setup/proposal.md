# Proposal: Split Workspace into Dedicated Repositories & CI/CD Workflows

## Why

The RadioKit project has evolved to contain multiple distinct ecosystems within a single directory tree:
1. An ESP32 / Arduino C++ embedded library (`rk-arduino`)
2. A Rust WebSocket Relay server (`radiokit-relay`)
3. A centralized OpenSpec specifications repository (`openspec`)
4. An Astro documentation and marketing website (`website`)
5. A Flutter companion app with visual UI designer and multi-platform build systems (`RadioKit`)

Maintaining these in a coupled tree complicates independent versioning, issue tracking, CI build triggers, and release pipelines (e.g., publishing the Arduino library to the Arduino Registry / PlatformIO, deploying the private relay to cloud servers, hosting the Astro website on GitHub Pages, and building multiplatform Flutter binaries).

Splitting them into dedicated repositories under the GitHub organization with tailored CI/CD workflows establishes clean boundaries and independent lifecycles.

## What Changes

- **Repository Separation**:
  - `RK-Arduino` (Public repo): Initialize standalone Git repository from `/home/sun/Apps/RCKIT/rk-arduino`, ignoring build artifacts (`.pio`, `.cache`, `compile_commands.json`).
  - `openspec` (Public repo): Initialize standalone Git repository from `/home/sun/Apps/RCKIT/openspec`, acting as the centralized OpenSpec store registered across workspaces.
  - `RadioKit-Relay` (Private repo): Initialize standalone Git repository from `/home/sun/Apps/RCKIT/radiokit-relay`, ignoring `target/` and temporary artifacts.
  - `website` (Public repo): Move `/home/sun/Apps/RCKIT/RadioKit/website` to `/home/sun/Apps/RCKIT/website`, initialize standalone Git repository, and ignore `node_modules/`, `.astro/`, `dist/`.
  - `RadioKit` (Public repo): Clean up and commit removal of extracted subdirectories, retaining Flutter app and multiplatform release workflows.
- **Workflow / CI Migration**:
  - Migrate and adjust `pioarduino-ci.yml` into `.github/workflows/pioarduino-ci.yml` inside `RK-Arduino`.
  - Migrate and adjust `relay-ci.yml` into `.github/workflows/relay-ci.yml` inside `RadioKit-Relay`.
  - Migrate and adjust `deploy-website.yml` into `.github/workflows/deploy-website.yml` inside `website`.
  - Keep Flutter app CI and multiplatform release workflows (`flutter-ci.yml`, `release-android.yml`, `release-ios.yml`, `release-linux-flatpak.yml`, `release-macos.yml`, `release-windows.yml`) inside `RadioKit`.
- **OpenSpec Store Registration**:
  - Register `/home/sun/Apps/RCKIT` as the central OpenSpec store (`radiokit`) so changes and specs can be queried or managed seamlessly.

## Capabilities

### New Capabilities
- `multirepo-organization`: Architecture, repository layout, `.gitignore` configurations, and Git initialization for each dedicated repo.
- `multirepo-ci-workflows`: Dedicated GitHub Actions workflow configurations for PlatformIO builds, Rust CI, Astro website deployment, and Flutter app builds.

### Modified Capabilities
<!-- None -->

## Impact

- **File System**: `website/` moved out of `RadioKit/` to `/home/sun/Apps/RCKIT/website/`.
- **Git Roots**: Five standalone Git repositories will be active under `/home/sun/Apps/RCKIT/`.
- **CI/CD**: Workflows in each repository trigger only on relevant codebase changes, drastically speeding up CI execution.
- **Tooling / OpenSpec**: All repositories reference the central `openspec` store.
