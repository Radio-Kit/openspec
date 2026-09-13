## ADDED Requirements

### Requirement: Standalone RK-Arduino Git Repository
The `rk-arduino` folder SHALL be initialized as a standalone Git repository named `RK-Arduino` with appropriate ignore rules for PlatformIO and C++ build artifacts.

#### Scenario: Git initialization and staging
- **WHEN** git is initialized in `/home/sun/Apps/RCKIT/rk-arduino`
- **THEN** it tracks C++ source files, headers, examples, `library.properties`, and `library.json`, while ignoring `.pio/`, `.cache/`, `compile_commands.json`, `.kmeta.yaml`, and `.vscode/`

### Requirement: Standalone RadioKit-Relay Git Repository
The `radiokit-relay` folder SHALL be initialized as a standalone Git repository named `RadioKit-Relay` with appropriate ignore rules for Rust build artifacts.

#### Scenario: Git initialization and staging
- **WHEN** git is initialized in `/home/sun/Apps/RCKIT/radiokit-relay`
- **THEN** it tracks Rust source files, `Cargo.toml`, `Cargo.lock`, `Dockerfile`, and tests, while ignoring `target/` and temporary artifacts

### Requirement: Standalone Website Repository
The website folder currently located at `/home/sun/Apps/RCKIT/RadioKit/website` SHALL be moved to `/home/sun/Apps/RCKIT/website` and initialized as a standalone Git repository named `website`.

#### Scenario: Relocation and git initialization
- **WHEN** the website directory is moved to `/home/sun/Apps/RCKIT/website` and git is initialized
- **THEN** it tracks Astro configuration, source components, and `package.json`, while ignoring `node_modules/`, `dist/`, and `.astro/`

### Requirement: Standalone OpenSpec Central Store
The `openspec` folder SHALL be maintained as a centralized specification store registered across repositories.

#### Scenario: Store registration
- **WHEN** `openspec store register` is executed for the OpenSpec root
- **THEN** the store `radiokit` is registered and accessible via `openspec --store radiokit`

### Requirement: Clean RadioKit Flutter Repository
The main `RadioKit` repository SHALL retain the Flutter companion application and its multiplatform build assets, removing tracked submodules that were extracted into dedicated repositories.

#### Scenario: Repository cleanup
- **WHEN** git status is inspected in `/home/sun/Apps/RCKIT/RadioKit`
- **THEN** extracted subdirectories are unlinked and clean, and the Flutter app remains intact
