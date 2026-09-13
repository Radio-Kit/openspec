# multirepo-ci-workflows Specification

## Purpose
TBD - created by archiving change split-multirepo-setup. Update Purpose after archive.
## Requirements
### Requirement: RK-Arduino PlatformIO CI Workflow
The `RK-Arduino` repository SHALL include a GitHub Actions workflow that compiles all example sketches across supported architectures (ESP32, STM32, RP2040) using root-relative paths.

#### Scenario: PlatformIO build execution
- **WHEN** commits or pull requests are pushed to `RK-Arduino`
- **THEN** the `pioarduino-ci.yml` workflow installs PlatformIO, caches packages, and builds all examples from `examples/<dir>` without `rk-arduino/` path prefixes

### Requirement: RadioKit-Relay Rust CI Workflow
The `RadioKit-Relay` repository SHALL include a GitHub Actions workflow that validates Rust code quality and runs unit/integration test suites.

#### Scenario: Rust CI execution
- **WHEN** commits or pull requests are pushed to `RadioKit-Relay`
- **THEN** the `relay-ci.yml` workflow runs `cargo fmt --check`, `cargo clippy -- -D warnings`, `cargo build`, and `cargo test` from the repository root

### Requirement: Website Astro Deployment Workflow
The `website` repository SHALL include a GitHub Actions workflow that builds the Astro site and deploys it to GitHub Pages.

#### Scenario: Website build and deployment
- **WHEN** commits are pushed to `main` in `website`
- **THEN** the `deploy-website.yml` workflow installs npm dependencies, builds the Astro project with `npm run build`, packages `dist/`, and publishes to GitHub Pages

### Requirement: RadioKit Flutter CI & Release Workflows
The `RadioKit` repository SHALL retain Flutter testing CI and multiplatform release build workflows while removing external repository build jobs.

#### Scenario: Flutter workflow execution
- **WHEN** changes are pushed to `RadioKit`
- **THEN** `flutter-ci.yml` and platform release workflows execute against `radiokit-app`, and obsolete Arduino/Relay/Website workflows are removed from `.github/workflows/`

