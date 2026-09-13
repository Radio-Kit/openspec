## ADDED Requirements

### Requirement: Unified single-pass compressed flashing
The flasher provider SHALL build a single contiguous binary image with 0xFF padding across all bundle partition parts and stream it in one compressed write operation.

#### Scenario: Single-pass flash write
- **WHEN** the user initiates flashing for a firmware release bundle
- **THEN** all partition parts from offset 0x0000 are written in a single uninterrupted deflate session
