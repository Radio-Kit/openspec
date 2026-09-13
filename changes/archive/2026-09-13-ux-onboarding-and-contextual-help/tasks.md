## 1. Help & Tour State Management

- [x] 1.1 Add tour flags (`hasSeenModelsTour`, `hasSeenFlasherTour`, `hasSeenDesignerTour`) and reset method to `SettingsProvider`
- [x] 1.2 Persist and restore tour flags from `SharedPreferences` in `SettingsProvider`
- [x] 1.3 Add a "Reset Help & Guides" button in `SystemTab` settings

## 2. Reusable Contextual Help System

- [x] 2.1 Create static help topic registry in `lib/widgets/help/help_content.dart` (Baud Rate, CDC Touch, LittleFS, Transports, Ed25519 pairing)
- [x] 2.2 Implement `HelpBottomSheet` with title, explanation, hardware tips, and dismiss action
- [x] 2.3 Implement `HelpBadge` UI component that triggers `HelpBottomSheet`

## 3. In-Situ Spotlight Coach Mark System

- [x] 3.1 Implement custom `SpotlightOverlay` with cutout painter, tooltip card, target focus, and next/skip controls
- [x] 3.2 Wire first-visit spotlight tour on `ModelsTab` (Scan button, Pair sheet trigger, Tab switcher)
- [x] 3.3 Wire first-visit spotlight tour on `FlasherTab` (Port selector, Firmware catalog, Flash button)
- [x] 3.4 Wire first-visit spotlight tour on `DesignerScreen` (Palette, Inspector, Play Mode toggle, C++ code export)

## 4. Hardware Discovery & Troubleshooting

- [x] 4.1 Implement actionable diagnostic checklist widget for empty discovery on `ModelsTab` (Bluetooth power, Location permissions, USB-OTG connection)
- [x] 4.2 Implement actionable empty state on `FlasherTab` for missing serial ports (USB data cable check, CH340/CP2102 drivers)
- [x] 4.3 Implement interactive manual bootloader sequence dialog (Hold BOOT -> Press EN -> Release BOOT)
- [x] 4.4 Embed `HelpBadge` beside key technical settings in `FlasherTab`, `ModelsTab`, and `DesignerScreen`

## 5. Verification & Testing

- [x] 5.1 Add unit tests for `SettingsProvider` tour flags and `resetHelpGuides`
- [x] 5.2 Add widget tests for `HelpBadge` and `HelpBottomSheet` rendering
- [x] 5.3 Verify first-run tours trigger once and do not re-trigger after dismissal or navigation
