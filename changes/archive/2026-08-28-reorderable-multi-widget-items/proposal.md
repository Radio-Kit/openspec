## Why

Currently, managing items for multi-button (`multiButton`) and multi-select (`multiSelect`) widgets in the visual designer inspector relies solely on an external `Items` numeric stepper. Adding or removing items only truncates or appends at the end of the list. If a user needs to reorder items or delete an item in the middle (e.g., removing item 2 out of 4), they are forced to manually rewrite text labels and re-pick icons for all subsequent items.

Making items directly draggable, reorderable, and deletable with inline action buttons makes editing multi-item widgets fast, intuitive, and error-free.

## What Changes

- **Drag-and-Drop Reordering**: Multi-widget items in the inspector can now be reordered via drag handles using `ReorderableListView`.
- **Inline Delete Action**: Each item card features a delete button to remove that specific item at any position (disabled when only 1 item remains).
- **Inline Add Action**: An `+ Add Item` button in the items header appends a new item up to the maximum limit (8 items).
- **Automatic Dimension Adjustment**: Adding or deleting items automatically recalculates and resizes the widget's canvas dimensions to maintain proper aspect ratio.
- **Removed Stepper**: Replaced the separate numeric stepper with a dynamic item counter badge and header action.

## Capabilities

### New Capabilities
- `multi-widget-item-reorder`: Drag-and-drop reordering, inline addition, inline deletion, and aspect-ratio synchronization for multi-button and multi-select inspector items.

### Modified Capabilities

## Impact

- `radiokit-app/lib/screens/designer/widgets/designer_inspector.dart`: Refactors `_DesignerMultiItemEditor` and related field builders.
- No breaking changes to the JSON schema or Arduino codegen; `items` array and `itemCount` integer in element properties remain fully compatible.
