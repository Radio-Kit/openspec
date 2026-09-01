## Context

The RadioKit ecosystem currently lives inside `/home/sun/Apps/RCKIT/` with multiple subcomponents:
- `rk-arduino`: Embedded Arduino C++ framework for ESP32/STM32/RP2040
- `radiokit-relay`: Axum/Tokio Rust relay server with Ed25519 auth & WebSocket streaming
- `openspec`: Central OpenSpec specifications and changes
- `RadioKit/website`: Astro + Tailwind documentation & landing page
- `RadioKit`: Flutter multiplatform companion application

We are separating them into 5 distinct repositories intended for publication to a GitHub organization, initializing fresh Git histories, tailoring .gitignore rules to omit build artifacts, migrating and tuning their corresponding GitHub Actions CI/CD workflows, and registering `openspec` as a central store.

## Goals / Non-Goals

**Goals:**
- Create clean, isolated Git repositories for `RK-Arduino`, `RadioKit-Relay`, `openspec`, `website`, and `RadioKit`.
- Relocate `RadioKit/website` to `/home/sun/Apps/RCKIT/website`.
- Configure robust `.gitignore` files for each repository so build output (`.pio`, `node_modules`, `.astro`, `target`, `compile_commands.json`) is not tracked.
- Distribute and adapt GitHub Actions workflow files to each repo's root context:
  - `pioarduino-ci.yml` in `RK-Arduino/.github/workflows/`
  - `relay-ci.yml` in `RadioKit-Relay/.github/workflows/`
  - `deploy-website.yml` in `website/.github/workflows/`
  - Flutter CI and multiplatform release workflows in `RadioKit/.github/workflows/`
- Clean up deleted files from `RadioKit` and maintain Flutter app functionality.
- Configure OpenSpec store registration across local workspaces.

**Non-Goals:**
- Rewriting historical Git commits (`git-filter-repo` / `git subtree` splitting) — clean fresh histories with `git init -b main` will be created.
- Altering core application logic, Arduino C++ code, or Rust server code during the split.

## Decisions

1. **Clean `git init` vs Subtree Split**:
   - *Decision*: Initialize fresh git repos with `git init -b main`.
   - *Rationale*: Confirmed by user; provides clean, lightweight initial commits for public and private repos without dragging historical intermediate commits or heavy binary history.

2. **Standalone Website Repository**:
   - *Decision*: Relocate `RadioKit/website` up to `/home/sun/Apps/RCKIT/website`.
   - *Rationale*: Decouples documentation / Astro build pipeline from Flutter app development.

3. **Workflow Path Adjustments**:
   - *Decision*: In `RK-Arduino`, update `working-directory: examples/${{ matrix.dir }}` and cache paths from `rk-arduino/**` to relative repo paths. In `RadioKit-Relay`, adjust working directory to repo root. In `website`, adjust paths to repo root.
   - *Rationale*: Eliminates redundant subdirectory path prefixes now that each component is at its own repo root.

4. **Central OpenSpec Store Registration**:
   - *Decision*: Keep `openspec` at `/home/sun/Apps/RCKIT/openspec` registered as store `radiokit`.
   - *Rationale*: Allows running `openspec` commands with `--store radiokit` across any repository workspace.

## Risks / Trade-offs

- **[Risk]** Unwanted build artifacts (e.g. `node_modules`, `.pio`, `target`) committed accidentally.
  → **Mitigation**: Create explicit `.gitignore` files before running `git add .` and verify staged files.
- **[Risk]** Relative paths broken in migrated GitHub Actions workflows.
  → **Mitigation**: Audit all step working directories, cache hashes, and checkout configs to match new repo root boundaries.

## Migration Plan

1. Create target directories and move `RadioKit/website` to `/home/sun/Apps/RCKIT/website`.
2. Configure `.gitignore` and `.github/workflows/` for `rk-arduino`, `radiokit-relay`, and `website`.
3. Initialize git and commit source files in each new repository.
4. Clean up `RadioKit` repo (commit deletion of extracted folders and workflows).
5. Verify OpenSpec store registration.
