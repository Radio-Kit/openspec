## Context

The Flasher Tab marketplace view discovers and lists GitHub release binaries for ESP32 boards. Previously, repository cards were expanded by default, raw filenames were rendered without structured hierarchy, and clicking "DOWNLOAD & FLASH" automatically initiated serial flashing without staging in the primary FIRMWARE section.

## Goals / Non-Goals

**Goals:**
- Default all repository cards to collapsed on initial page load.
- Filter out single-partition OTA binaries (`*-ota.bin`) from the Flasher list, showing only flashable images.
- Structure firmware binary cards with Board Name as header, Chip family tag, Variant tag, size, and secondary asset filename.
- Rename action button to "SELECT FIRMWARE", downloading the binary and staging it into `FlasherProvider` without auto-starting flash.
- Remove `-factory` suffix in `RC_brain/scripts/package_release.py`.

**Non-Goals:**
- Modifying the wireless OTA update flow.
- Changing GitHub release API schemas or endpoints.

## Decisions

### Decision 1: Collapsed State by Default
- **Choice**: Track expanded repository URLs in a `Set<String> _expandedCards = {}` in `_MarketplaceSectionState`.
- **Rationale**: Keeps the page clean, compact, and scannable.

### Decision 2: Distinct Board, Variant, and Chip Fields in `MarketplaceBinaryInfo`
- **Choice**: Add `board` and `variant` properties to `MarketplaceBinaryInfo`, with a fallback `displayName` getter.
- **Rationale**: Allows consistent rendering of `Board`, `Chip`, and `Variant` across any repository release structure.

### Decision 3: "SELECT FIRMWARE" Staging Workflow
- **Choice**: When "SELECT FIRMWARE" is clicked, download the binary to a temporary file, call `flasher.setSelectedFirmwareDirect(name, path, bytes)`, and show a confirmation snackbar directing the user to the `FIRMWARE` section above.
- **Rationale**: Decouples downloading from immediate flashing, giving users full visibility and control before flashing over serial.

## Risks / Trade-offs

- **[Risk] Binaries without parsed board name**:
  - **Mitigation**: `displayName` falls back to `project` name or asset filename without extension if board token is unavailable.
