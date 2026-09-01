## Context

Multi-button (`multiButton`) and multi-select (`multiSelect`) widgets in RadioKit Designer allow users to configure segmented buttons with custom text and icons for ON and OFF states. Previously, adjusting item counts was done using a generic numeric stepper (`_buildMultiItemCountField`), which truncated or appended items at the tail of the list. Rearranging items or removing an item in the middle required manual re-entry of all subsequent items.

## Goals / Non-Goals

**Goals:**
- Provide direct drag-and-drop reordering of items within `_DesignerMultiItemEditor`.
- Allow deleting any item at an arbitrary index with a single click.
- Allow adding new items with an inline `+ Add` button in the section header.
- Automatically synchronize element canvas dimensions (aspect ratio) when items are added or deleted.
- Replace the separate numeric stepper with a header counter (`ITEMS (N/8)`) and actions.
- Maintain full compatibility with existing JSON schemas, serialization, and Arduino code generation.

**Non-Goals:**
- Altering the widget serialization structure or Arduino code generation format.
- Collapsing cards into an accordion (kept clean and expanded per user preference).

## Decisions

### Decision 1: Use `ReorderableListView` with Custom Drag Handle
- **Choice**: Use Flutter's `ReorderableListView.builder` (or `ReorderableListView`) configured with `shrinkWrap: true`, `physics: const NeverScrollableScrollPhysics()`, `buildDefaultDragHandles: false`, and `ReorderableDragStartListener` around a dedicated drag handle icon (`PhosphorIconsRegular.dotsSixVertical`).
- **Rationale**: Keeps drag initiation intentional and avoids interfering with text field editing or scrolling inside the inspector panel.

### Decision 2: Batch State Updates and Auto-Resizing in Editor
- **Choice**: Encapsulate `_addItem`, `_deleteItem`, and `_reorderItems` inside `_DesignerMultiItemEditorState` or helper callbacks that update `DesignerState` properties (`items`, `itemCount`) and recalculate width/height via `DesignerState.updateElementSize`.
- **Rationale**: Keeps all multi-item management logic coherent and cleanly responsive without needing separate redundant number controls.

## Risks / Trade-offs

- **[Risk] Nested scroll conflicts**: `ReorderableListView` inside a scrollable inspector column might cause scroll collisions.
  - **Mitigation**: Set `physics: const NeverScrollableScrollPhysics()` and `shrinkWrap: true` on `ReorderableListView`.
- **[Risk] Key collision in Flutter reorderable lists**:
  - **Mitigation**: Provide stable `ValueKey` based on unique item identifier or position indices during builds.
