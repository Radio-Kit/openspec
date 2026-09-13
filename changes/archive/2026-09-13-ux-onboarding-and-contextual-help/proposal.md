## Why

RadioKit currently provides extensive embedded capabilities (BLE, USB Serial, WiFi, firmware flashing, and visual dashboard design), but new users face a steep learning curve upon launching the app. Without an active, pre-flashed ESP32 nearby, users land on an unguided screen without clear indications of next steps, how hardware discovery operates, or what embedded configuration settings mean. 

Rather than interrupting the workflow with full-screen onboarding slides, RadioKit needs an unobtrusive, hardware-focused guidance system: in-situ spotlight coach marks on first visit to primary workflows, standardized contextual `(?)` explanation sheets for embedded settings, and actionable hardware discovery troubleshooting when no devices or serial ports are detected.

## What Changes

- **In-Situ Spotlight Coach Marks**: Lightweight, first-visit overlay coach marks pointing out core actions across key screens:
  - Models Tab: Highlights device scanning, pairing actions, and navigation.
  - Flasher Tab: Highlights port selection, firmware catalog, and flash trigger.
  - Designer Screen: Highlights canvas drag-and-drop, Inspector configuration, Play Mode toggle, and C++ code export.
  - Persistent state in settings with a "Reset Help & Guides" action to re-trigger tours.
- **Contextual Help System**: Standardized `HelpBadge` widget and `HelpBottomSheet` providing clear, jargon-free explanations and hardware tips beside technical parameters (e.g., Baud Rate, 1200-baud CDC touch, LittleFS partition format, Ed25519 pairing key).
- **Physical Hardware Diagnostic Empty States**: Actionable empty states on Models and Flasher tabs providing diagnostic checklists (BLE permissions, USB OTG data cable verification, ESP32 BOOT+RESET bootloader button guide).

## Capabilities

### New Capabilities
- `in-situ-spotlight-tour`: First-visit overlay coach marks highlighting essential interactions on Models, Flasher, and Designer screens, with persistence and reset controls.
- `contextual-help-system`: Reusable `(?)` badge and modal bottom sheet explaining embedded technical settings and best practices across the application.
- `hardware-discovery-troubleshooting`: Actionable diagnostic checklists and manual bootloader guides for physical ESP32 USB and BLE connectivity.

### Modified Capabilities
<!-- None: Core protocols, flashing engine, and widget codegen requirements remain unchanged. -->

## Impact

- **Flutter Companion App** (`radiokit-app`):
  - Add new UI widgets in `lib/widgets/help/` (`help_badge.dart`, `help_bottom_sheet.dart`, `spotlight_tour.dart`).
  - Update `SettingsProvider` to manage tour completion flags and reset actions.
  - Integrate contextual help badges and diagnostic empty states into `models_tab.dart`, `flasher_tab.dart`, and `designer_screen.dart`.
  - Dependencies: Utilizes Flutter's custom painters or lightweight overlay mechanics without adding heavyweight external dependencies.
