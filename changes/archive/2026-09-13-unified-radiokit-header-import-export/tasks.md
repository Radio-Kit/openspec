## 1. Header Generator & Regex Normalization

- [x] 1.1 Add `generateFullHeader(Map<String, dynamic> json)` to `JsonArduinoGenerator` in `radiokit-app/lib/screens/designer/codegen/json_arduino_generator.dart`
- [x] 1.2 Update `DesignerState.configPattern` and `header_file_parser.dart` to support both `RADIOKIT_Designer_Config` and `RadioKit_UI_Designer_Config` case-insensitively
- [x] 1.3 Update `designer_screen.dart` to use `JsonArduinoGenerator.generateFullHeader`

## 2. Remote Access Service Import/Export Endpoints

- [x] 2.1 Update `_handleDesignHeader` in `remote_access_service.dart` to return `JsonArduinoGenerator.generateFullHeader(json)`
- [x] 2.2 Add `POST /api/designs/import` route and update `_handleDesignsSave` in `remote_access_service.dart` to accept raw JSON, full `.h` files, and base64 payloads
- [x] 2.3 Add error handling for corrupted `.h` files or invalid JSON payloads

## 3. Unit Tests & Verification

- [x] 3.1 Add unit tests for `generateFullHeader` and round-trip parsing from `.h` string
- [x] 3.2 Add unit tests for `POST /api/designs/import` with `.h` header and `.json` payload
- [x] 3.3 Verify all Flutter tests pass
