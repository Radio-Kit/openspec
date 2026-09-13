# multi-part-flashing Specification

## Purpose
TBD - created by archiving change reliable-esp32-multi-part-flasher. Update Purpose after archive.
## Requirements
### Requirement: Sequential Multi-Part Flashing
The firmware flasher SHALL flash each partition defined in a firmware bundle's manifest independently at its specified flash offset without inserting contiguous `0xFF` padding between sparse partitions.

#### Scenario: Flashing multi-part bundle with sparse partitions
- **WHEN** a firmware bundle with multiple parts (e.g. bootloader at 0x0, partitions at 0x8000, otadata at 0xe000, app at 0x10000, optional littlefs at 0x310000) is flashed
- **THEN** the flasher writes each part sequentially to its exact offset and reports aggregate progress across all parts

### Requirement: Aggregate Multi-Part Progress Reporting
The flasher provider SHALL compute and emit a continuous progress fraction representing the total bytes written across all parts relative to the total binary bytes of the bundle.

#### Scenario: Progress update during multi-part flash
- **WHEN** a partition part is being flashed
- **THEN** the UI progress indicator reflects the weighted percentage of the total bundle size written so far

### Requirement: Synchronous Android USB Transport Write
The `FlserialPortAdapter` on Android SHALL await USB write completion before resolving write futures to prevent buffer overruns and inaccurate completion status.

#### Scenario: High-speed flash payload transmission on Android
- **WHEN** flash blocks are transmitted over USB serial
- **THEN** each block write completes only after the underlying Android USB bulk transfer has completed

