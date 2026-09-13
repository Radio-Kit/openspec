## ADDED Requirements

### Requirement: Full Header Generation with Embedded Designer JSON
`JsonArduinoGenerator` SHALL provide a method `generateFullHeader(Map<String, dynamic> json)` that returns the complete `RADIOKIT.h` containing:
1. The standard comment block `/*__RADIOKIT_Designer_Config__\n<formatted-json>\nRADIOKIT_Designer_Config__*/`
2. The generated C++ declarations, pin mappings, and widget instances.

`GET /api/designs/<id>/header` SHALL return the output of `generateFullHeader` as `text/plain; charset=utf-8`.

#### Scenario: Generate full header from design JSON
- **WHEN** `generateFullHeader` is called with a design JSON
- **THEN** it returns complete C++ source with embedded config comments and valid declarations

### Requirement: Multi-Format Design Import
`POST /api/designs/import` and `POST /api/designs` SHALL support:
1. **Raw JSON**: An object containing `{ "version": ..., "config": ..., "pages": [...] }`.
2. **C++ Header String**: Text containing `/*__RADIOKIT_Designer_Config__ ... */` or `/*__RadioKit_UI_Designer_Config__ ... */`.
3. **Base64 Payload**: Encoded string containing either format.

The endpoint SHALL extract the visual designer JSON, persist it with `DesignsProvider`, and return HTTP 200 with design metadata.

#### Scenario: Import design from header text
- **WHEN** a client POSTs a C++ header containing embedded designer config
- **THEN** the server extracts the config JSON and saves the design successfully
