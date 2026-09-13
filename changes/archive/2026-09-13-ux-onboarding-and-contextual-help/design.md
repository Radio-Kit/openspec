## Context

RadioKit is a companion tool for ESP32 microcontrollers supporting USB Serial, BLE, WiFi, and Cloud transports, alongside an in-app visual Designer and Firmware Flasher. Currently, the user interface provides little inline guidance for complex embedded configurations (such as baud rates, USB CDC 1200-baud touching, LittleFS flash layouts, and hardware bootloader modes). First-time users landing on empty screens lack clear visual direction on how to interact with real hardware.

To address this without cumbersome multi-step slide carousels, this design introduces:
1. An in-situ spotlight coach-mark overlay system for first-time screen visits.
2. A standardized `HelpBadge` and `HelpBottomSheet` for inline contextual explanations.
3. Diagnostic hardware empty states with actionable troubleshooting checklists and an interactive BOOT/RESET bootloader sequence guide.

## Goals / Non-Goals

**Goals:**
- Provide zero-interruption, lightweight spotlight coach marks highlighting core actions on first visit to Models, Flasher, and Designer screens.
- Keep tour states persisted in `SettingsProvider` / `SharedPreferences` with a clear user option to reset guides.
- Implement reusable contextual `HelpBadge` and `HelpBottomSheet` widgets integrated next to complex technical settings.
- Build actionable empty states and an interactive ESP32 bootloader guide for physical hardware connection issues.

**Non-Goals:**
- Full-screen multi-page slide introductory carousels.
- Simulated or virtual mock devices (guidance remains strictly focused on physical hardware).
- Introducing heavy external coach-mark dependencies when Flutter's native layout and custom paint overlays suffice.

## Decisions

### 1. In-Situ Overlay Architecture
- **Decision**: Implement a lightweight, custom `SpotlightTour` widget using Flutter's `OverlayEntry` and `CustomPainter` with a cutout path (`Path.combine` with `PathOperation.difference`).
- **Rationale**: Avoids adding unmaintained third-party packages or risking version incompatibilities across Flutter web, desktop, and mobile. Allows precise theme matching (colors, fonts, and dark mode tokens).
- **Alternatives considered**: Third-party packages like `tutorial_coach_mark` or `showcaseview`. Rejected due to additional dependency overhead and rigid layout constraints on responsive tablets/landscape.

### 2. State & Persistence in SettingsProvider
- **Decision**: Track `hasSeenModelsTour`, `hasSeenFlasherTour`, and `hasSeenDesignerTour` flags in `SettingsProvider`, serialized to `SharedPreferences`. Provide `resetHelpGuides()` to clear these flags.
- **Rationale**: Centralizes user preferences alongside existing settings (`useFullscreen`, `overrideTheme`), ensuring seamless reactivity with Provider.

### 3. Centralized Help Definitions (`HelpContent`)
- **Decision**: Define static, strongly typed help topics in `lib/widgets/help/help_content.dart` containing title, summary, technical rationale, and troubleshooting tip.
- **Rationale**: Keeps help text maintainable, localized in one place, and easy to review or update without cluttering presentation widgets.

### 4. Interactive Bootloader Sequence Helper
- **Decision**: Create an animated or step-by-step interactive visual sheet showing the ESP32 `BOOT` and `EN/RST` button sequence (Hold BOOT -> Click EN -> Release BOOT).
- **Rationale**: ESP32 download mode failure is the single most common failure point for physical hardware flashing, especially on boards lacking auto-reset circuits.

## Risks / Trade-offs

- **[Risk] Screen layout changes or animations causing spotlight cutout offset** → *Mitigation*: Calculate target `GlobalKey` bounding boxes dynamically when the overlay mounts and on layout/orientation changes.
- **[Risk] User dismisses tour accidentally and misses core information** → *Mitigation*: Provide a "Reset Help & Guides" button in System Settings, and ensure every key action is independently discoverable via contextual `HelpBadge` widgets.
