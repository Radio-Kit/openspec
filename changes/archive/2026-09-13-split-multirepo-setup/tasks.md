## 1. Relocate Website

- [x] 1.1 Move `/home/sun/Apps/RCKIT/RadioKit/website` to `/home/sun/Apps/RCKIT/website`
- [x] 1.2 Add `.gitignore` to `/home/sun/Apps/RCKIT/website` ignoring `node_modules/`, `dist/`, `.astro/`, `.env*`
- [x] 1.3 Create `.github/workflows/deploy-website.yml` in `website` configured with root-relative paths
- [x] 1.4 Initialize Git in `/home/sun/Apps/RCKIT/website` and create initial commit

## 2. Setup RK-Arduino Repository

- [x] 2.1 Add `.gitignore` to `/home/sun/Apps/RCKIT/rk-arduino` ignoring `.pio/`, `.cache/`, `compile_commands.json`, `.kmeta.yaml`, `.vscode/`
- [x] 2.2 Create `.github/workflows/pioarduino-ci.yml` in `rk-arduino` with root-relative example paths
- [x] 2.3 Initialize Git in `/home/sun/Apps/RCKIT/rk-arduino` and create initial commit

## 3. Setup RadioKit-Relay Repository

- [x] 3.1 Add `.gitignore` to `/home/sun/Apps/RCKIT/radiokit-relay` ignoring `target/` and `.env`
- [x] 3.2 Create `.github/workflows/relay-ci.yml` in `radiokit-relay` with root-relative cargo commands
- [x] 3.3 Initialize Git in `/home/sun/Apps/RCKIT/radiokit-relay` and create initial commit

## 4. Setup OpenSpec Central Store

- [x] 4.1 Add `.gitignore` to `/home/sun/Apps/RCKIT/openspec` (if needed)
- [x] 4.2 Initialize Git in `/home/sun/Apps/RCKIT/openspec` and create initial commit
- [x] 4.3 Verify local store registration via `openspec store list`

## 5. Clean & Update RadioKit (Flutter App) Repository

- [x] 5.1 Remove obsolete CI workflows (`deploy-website.yml`, `pioarduino-ci.yml`, `relay-ci.yml`) from `RadioKit/.github/workflows/`
- [x] 5.2 Commit unlinked subdirectories and workflow updates in `RadioKit`
- [x] 5.3 Verify Flutter project builds / test passes cleanly in `RadioKit`
