## MODIFIED Requirements

### Requirement: Firmware Reports Version in Device Info Frame
The firmware `RadioKitClass::_handleSettingsDeviceInfo()` handler SHALL append both the board identifier string (`RadioKit.config.board`) and firmware version string (`RadioKit.config.version`) to the `SETTINGS_RESP_DEVICE_INFO_DATA` (0x88) payload.

#### Scenario: Host queries device info
- **WHEN** the companion app sends `SETTINGS_CMD_GET_DEVICE_INFO` (0x08)
- **THEN** the firmware returns a payload containing `[PROTO_VER][NAME_LEN][NAME][DESC_LEN][DESC][UID_LEN][UID][ICON_LEN][ICON][BOARD_LEN][BOARD][VER_LEN][VERSION]`
- **AND** the companion app parses and stores `board` and `firmwareVersion` strings in `DeviceProvider`.

### Requirement: Companion App Matches Binary Assets
The companion app SHALL filter release binary assets for `-ota` suffix by default when present, and match candidate `.bin` firmware assets against the connected device's board identifier.

#### Scenario: Release contains `-ota` assets matching board identifier
- **WHEN** the connected device board is `TRACKLINK_V3` and the release assets contain `TRACKLINK_V3-ota.bin`, `TRACKLINK_V3-full.bin`, and `GTRACK_V1-ota.bin`
- **THEN** only `-ota` assets are shown by default and `TRACKLINK_V3-ota.bin` is automatically selected as the target asset.

#### Scenario: User toggles Show All binaries
- **WHEN** `-ota` filtering is active and the user clicks the "SHOW ALL" toggle
- **THEN** all `.bin` assets in the release are made available in the asset selector dropdown.

#### Scenario: Release has no `-ota` naming convention
- **WHEN** none of the `.bin` assets contain `-ota` or `_ota` in their filenames
- **THEN** all `.bin` assets are shown without filtering and the best match against the device's board identifier is selected.
