# Design: Unified RADIOKIT.h Header Generation and Multi-Format Project Import

## Architecture

```
                          ┌────────────────────────────────────────────────────────┐
                          │               Import Project Payload                   │
                          │   (Raw JSON or complete RADIOKIT.h / Base64 / String)  │
                          └───────────────────────────┬────────────────────────────┘
                                                      │
                                                      ▼
                                       ┌─────────────────────────────┐
                                       │     Format Auto-Detector    │
                                       └──────────────┬──────────────┘
                                                      │
                                  ┌───────────────────┴───────────────────┐
                                  ▼                                       ▼
                       ┌──────────────────────┐               ┌──────────────────────┐
                       │   JSON Object / Map  │               │   C++ Header (.h)    │
                       └──────────┬───────────┘               └──────────┬───────────┘
                                  │                                      │
                                  │                                      │ Extract Comment Block:
                                  │                                      │ /*__RADIOKIT_Designer_Config__
                                  │                                      │ ...
                                  │                                      │ RADIOKIT_Designer_Config__*/
                                  │                                      ▼
                                  │                            ┌──────────────────────┐
                                  │                            │  Parsed JSON Config  │
                                  │                            └──────────┬───────────┘
                                  │                                       │
                                  └───────────────────┬───────────────────┘
                                                      ▼
                                       ┌─────────────────────────────┐
                                       │      DesignsProvider        │
                                       │    (Save & Persist DB)      │
                                       └──────────────┬──────────────┘
                                                      │
                                                      ▼
                                       ┌─────────────────────────────┐
                                       │   GET /api/designs/<id>/    │
                                       │          header             │
                                       └──────────────┬──────────────┘
                                                      │
                                                      ▼
                                       ┌─────────────────────────────┐
                                       │  Full Self-Contained Header │
                                       │   /*__JSON__*/ + C++ Code   │
                                       └─────────────────────────────┘
```

## 1. Header Generator Updates (`json_arduino_generator.dart`)
- Define `kHeaderConfigStart = '/*__RADIOKIT_Designer_Config__';` and `kHeaderConfigEnd = 'RADIOKIT_Designer_Config__*/';`
- Add `JsonArduinoGenerator.generateFullHeader(Map<String, dynamic> json)`:
  - Formats JSON with 2-space indentation.
  - Combines `$kHeaderConfigStart\n$jsonString\n$kHeaderConfigEnd\n\n$cppCode`.

## 2. Universal Config Pattern Regex
- Match both uppercase and legacy casing:
  ```dart
  static final RegExp configPattern = RegExp(
    r'/\*__(?:RADIOKIT|RadioKit_UI)_Designer_Config__(.*?)(?:RADIOKIT|RadioKit_UI)_Designer_Config__\*/',
    dotAll: true,
    caseSensitive: false,
  );
  ```

## 3. Remote API Endpoints (`remote_access_service.dart`)
- **`GET /api/designs/<id>/header`**:
  - Uses `JsonArduinoGenerator.generateFullHeader(json)` to return the complete `.h` file with embedded JSON.
- **`POST /api/designs/import` & `POST /api/designs`**:
  - Accepts body with `{ "content": "..." }`, `{ "data": "<base64>" }`, or direct raw JSON map.
  - Detects if content contains C++ header markers, extracts the embedded JSON, parses it, validates schema, and saves to `DesignsProvider`.
  - Returns `{ "ok": true, "id": "...", "name": "...", "pagesCount": N, "widgetsCount": M }`.
