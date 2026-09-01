## 1. Inspector Multi-Item Component Refactoring

- [x] 1.1 Update `_DesignerMultiItemEditor` in `lib/screens/designer/widgets/designer_inspector.dart` to support ReorderableListView with drag handles
- [x] 1.2 Add inline item deletion with min-count guard (count > 1) and dimension auto-resizing
- [x] 1.3 Add header "+ Add Item" button with max-count guard (count < 8), auto-labeling, and dimension auto-resizing
- [x] 1.4 Remove separate `_buildMultiItemCountField` number stepper from inspector fields for `multiButton` and `multiSelect`

## 2. Verification and Testing

- [x] 2.1 Run flutter analyze and tests to verify there are no compilation or regression errors
- [x] 2.2 Validate drag-and-drop reordering, deletion, and addition behaviors
