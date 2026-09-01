## Context

When connecting to a RadioKit device via BLE, the companion app sends `GET_FEATURES` (settings protocol 0xDD, sub 0x03) to determine which capabilities the device supports. The firmware responds with a bitmask indicating OTA, FS, WiFi, Cloud, BLE, password state, and print stream support. This bitmask controls which tabs appear in the config bottom sheet.

The current implementation has three problems:

1. **Silent failure**: `_requestFeatures()` swallows all errors with bare `catch (_)`, making it impossible to diagnose why the bitmask is 0.
2. **Aggressive timeout**: A2s timeout fires-and-forgets without retry. Five concurrent requests after connection can overwhelm the BLE TX queue.
3. **Invisible auth gating**: When a user authenticates with a user-level password, `isUserMode = true` hides FS/OTA tabs by design, but the UI provides no explanation.

The firmware double `startBLE()` issue is a code quality problem but not the root cause of missing tabs — `enableOTA()` and `enableFS()` set member variables that survive NimBLE reinitialization.

## Goals / Non-Goals

**Goals:**
- Make the features request path observable (log failures, timeouts, and received bitmask values)
- Add retry logic to recover from transient BLE timeouts
- Show a visual indicator when FS/OTA tabs are hidden due to user-level auth
- Add missing user password field to designer inspector and codegen
- Make device password field always visible in designer (not hidden when empty)
- Remove the duplicate `startBLE()` call from generated firmware code

**Non-Goals:**
- Changing the features bitmask protocol or firmware-side feature detection
- Changing the auth model (user vs device level)
- Redesigning the tab layout or adding new tabs
- Fixing the NimBLE double-init (the firmware codegen fix is separate from this change)

## Decisions

### Decision 1: Add logging and retry to `_requestFeatures()`

**Choice**: Log all failure modes (write error, timeout, parse error) and retry once after 500ms delay on timeout.

**Rationale**: The current silent catch makes debugging impossible. Adding logging at each failure point will immediately reveal the root cause on any device. A single retry after 500ms gives the BLE TX queue time to drain from the concurrent requests.

**Alternatives considered**:
- *Increase timeout to 5s*: Doesn't address the root cause (write failure) and delays the success path.
- *Remove fire-and-forget and await sequentially*: Would add latency to every connection. Not worth it for a feature that rarely changes.
- *Add a features-specific queue*: Overengineering for a single request/response pair.

### Decision 2: Show lock icon when `isUserMode` hides tabs

**Choice**: In `_DeviceInfoTabsState.build()`, when `isUserMode && (hasFs || hasOta)`, show a subtle lock icon with tooltip "Device-level access required for FS/OTA" next to the tab bar.

**Rationale**: Users connecting with user-level auth currently see no explanation for missing tabs. A non-intrusive indicator communicates the reason without disrupting the UI.

**Alternatives considered**:
- *Show tabs as disabled/greyed*: Clutters the UI with non-functional elements.
- *Show a banner above tabs*: Too prominent for a known-by-design restriction.
- *Add a toast/snackbar*: Temporary and easy to miss.

### Decision 3: Remove duplicate `startBLE()` from codegen template

**Choice**: Remove the `RadioKit.startBLE()` call from `initRadioKit()` in the generated `RADIOKIT.h` template, keeping only the one in `setup()`.

**Rationale**: `initRadioKit()` is called from `setup()`, and `setup()` also calls `startBLE()` separately. The double call reinitializes NimBLE unnecessarily during boot. The `begin()` call in `initRadioKit()` already calls `enableOTA()` and `enableFS()`, so only the BLE initialization needs to be in `setup()`.

**Alternatives considered**:
- *Remove from setup() and keep in initRadioKit()*: The codegen template has `initRadioKit()` defined in the header and `setup()` in the .ino file. Users may add their own code between `initRadioKit()` and `startBLE()` in setup. Better to keep the explicit call in setup.
- *Add a guard in `startBLE()` to prevent double-init*: Adds complexity to the library for a codegen problem.

### Decision 4: Password fields in MODEL section, conditional user password

**Choice**: Add device password and user password fields to the MODEL section of the designer inspector. The device password field is always visible. The user password field is only visible when the device password is non-empty.

**Rationale**: The firmware supports user passwords (`_nvsUserPwd`), but the user password is meaningless without a device password (the firmware validates this). Showing the user password field only when a device password is set communicates this dependency naturally. Placing both in MODEL keeps password configuration close to the device identity.

**Alternatives considered**:
- *Always show both fields*: Confusing for users who don't understand the dependency. The user password field would appear to do nothing if device password is empty.
- *Keep passwords only in settings tab*: Forces users to flash firmware first, then set passwords via the app. Poor UX for initial setup.
- *Put passwords in a separate SECURITY section*: Over-engineering for two fields.

### Decision 5: Move remote links into FEATURES section

**Choice**: Move FS URL and OTA URL fields from the standalone REMOTE LINKS section into the FEATURES section, conditionally shown under their respective feature toggles. FS URL appears only when "Enable Filesystem" is toggled on. OTA URL appears only when "Enable OTA" is toggled on.

**Rationale**: Remote links are feature-specific configuration — FS URL is only relevant when filesystem is enabled, OTA URL only when OTA is enabled. Placing them under their toggles makes the dependency obvious and declutters the inspector. The REMOTE LINKS section is removed entirely.

**Alternatives considered**:
- *Keep REMOTE LINKS as a separate section*: Separates related config from its feature toggle, making it unclear why the fields exist.
- *Show all fields always*: Shows FS URL even when filesystem is disabled, which is confusing.

## Risks / Trade-offs

- **[Retry adds 500ms to success path when first attempt fails]** → Acceptable. The retry only fires on timeout, not on success. Most connections will succeed on the first attempt.
- **[Lock icon may confuse users who don't understand auth levels]** → Mitigate with a tooltip that says "Connect with device password for full access".
- **[Removing duplicate startBLE() may break existing user firmware]** → Only affects newly generated code. Existing compiled firmware is unaffected. Users who have customized their setup() and rely on the double call will need to remove one themselves.
- **[User password field hidden when device password empty may confuse users]** → Add helper text "Set device password first" near the user password field area when it's hidden.
- **[Moving remote links into features may break existing workflow]** → The fields are functionally identical, just relocated. Users who previously set FS/OTA URLs will find them in the same inspector, just under the feature toggles.
- **[Logging adds minor console output]** → Use `ConsoleLogLevel.info` so it's visible but not prominent. Can be toggled off in debug settings.

## Migration Plan

1. Deploy Flutter app changes first (logging, retry, lock icon) — backward compatible, no firmware changes needed.
2. Regenerate firmware codegen templates — new projects get single `startBLE()`. Existing projects unaffected.
3. No rollback needed — changes are additive (logging) or UI-only (lock icon).
