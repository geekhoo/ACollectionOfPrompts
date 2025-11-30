# Plan: Inline Grouped RowSpan Mode for dxDataGrid

## Overview

Introduce a new grouping display mode (`inlineMerged`) that presents grouped data as a **flat list of data rows** with **vertically merged cells** for grouped columns. Unlike the default `headerRows` mode:

- **No group header rows** are inserted (`rowType: 'group'` items are not created)
- **No expand/collapse UI** — all data rows are always visible
- **Grouped columns** appear in the data row at their natural position (or reordered to the left) with `rowSpan` applied to visually merge cells sharing the same group value
- **Multi-level grouping** is hierarchical: outer groups span the full count of their descendants; inner groups span only within their parent's boundary

---

## Visual Example

**Data (3 people):**

| Name  | Country | Gender |
|-------|---------|--------|
| Alice | USA     | Female |
| Bob   | USA     | Male   |
| Carol | USA     | Female |

**Grouped by Country → Gender (inlineMerged mode):**

| Country           | Gender             | Name  |
|-------------------|--------------------|-------|
| USA (rowSpan=3)   | Female (rowSpan=2) | Alice |
|                   |                    | Carol |
|                   | Male (rowSpan=1)   | Bob   |

- Country "USA" cell spans 3 rows (all people from USA)
- Gender "Female" cell spans 2 rows (Alice + Carol)
- Gender "Male" cell spans 1 row (Bob)
- Non-grouped columns (Name) render normally per row

---

## 1. Configuration Options

### 1.1 Grid-Level Option

```ts
grouping: {
  displayMode: 'headerRows' | 'inlineMerged'  // default: 'headerRows'
}
```

| Value | Behavior |
|-------|----------|
| `headerRows` | Current default: inserts collapsible group header rows with expand/collapse UI |
| `inlineMerged` | New mode: flat data rows with rowSpan-merged grouped column cells; no group headers |

### 1.2 Column-Level Option (Optional Enhancement)

```ts
columns: [
  { 
    dataField: 'Country', 
    groupIndex: 0,
    showWhenGrouped: true,        // required for column to appear in data rows
    mergeGroupedCells: true       // opt-in/out of rowSpan merging (default: true when inlineMerged)
  }
]
```

- `mergeGroupedCells: false` allows a grouped column to appear in every row without spanning (useful for inline editing scenarios)

### 1.3 Mode Constraints

When `displayMode === 'inlineMerged'`:

| Option | Behavior |
|--------|----------|
| `grouping.autoExpandAll` | Ignored (no expand/collapse concept) |
| `grouping.expandMode` | Ignored |
| `grouping.allowCollapsing` | Ignored |
| `showWhenGrouped` | Required `true` for grouped columns to appear and receive rowSpan |
| Group summaries | Still calculated and can be rendered in the merged cell or a footer |

---

## 2. Data Processing: Flatten Grouped Items

**Location:** `GroupingDataControllerExtender` (`packages/devextreme/js/__internal/grids/data_grid/grouping/m_grouping.ts`)

### 2.1 Override `_beforeProcessItems(items)`

When `displayMode === 'inlineMerged'`:

1. **Do NOT call** `_processGroupItems()` (which injects `rowType: 'group'` items)
2. **Call new helper:** `_flattenGroupedItemsInline(items, groupsCount, groupColumns)`

### 2.2 New Helper: `_flattenGroupedItemsInline()`

**Purpose:** Traverse hierarchical grouped data and produce a flat array of `rowType: 'data'` items, annotated with group path metadata.

**Input:**

- `items`: Hierarchical grouped structure from DataSource (items with nested `.items` arrays)
- `groupsCount`: Number of group levels (e.g., 2 for Country → Gender)
- `groupColumns`: Array of group column configurations

**Output:** Flat array where each item has:

```ts
item.rowType = 'data';
item._groupPath = ['USA', 'Female'];          // Values at each group level
item._groupLevelBoundaries = [0, 1, 2, ...];  // Row indices where each level's group starts
```

**Algorithm:**

```
function flattenGroupedItemsInline(items, level, path, result):
  if level === groupsCount:
    // Leaf level: items are data rows
    for each item in items:
      item.rowType = 'data'
      item._groupPath = [...path]
      result.push(item)
  else:
    // Group level: items have .key and .items
    for each group in items:
      path[level] = group.key
      flattenGroupedItemsInline(group.items, level + 1, path, result)
```

---

## 3. RowSpan Computation

**Location:** Same controller, in `_afterProcessItems()` or within `_updateItemsCore`

### 3.1 When to Compute

Only when `displayMode === 'inlineMerged'` and at least one grouped column has `showWhenGrouped: true`.

### 3.2 Algorithm: Segment-Based RowSpan Calculation

For each group level `L` (from outermost to innermost):

1. **Resolve target column:** Find the visible column index for this group level via `_columnsController.getVisibleColumns()`

2. **Track segments:** Walk the flat items array and identify contiguous segments where `item._groupPath[L]` is constant

3. **Apply rowSpan metadata:**
   - At segment start (row `S`): `items[S]._rowSpan[visibleColIndex] = segmentLength`
   - At segment continuation (rows `S+1` to `S+N-1`): `items[i]._rowSpanSkip[visibleColIndex] = true`

4. **Hierarchical nesting:** When a higher-level segment closes, all nested segments also close (reset their tracking)

**Pseudocode:**

```
for each groupLevel L from 0 to groupsCount-1:
  visibleColIndex = getVisibleColumnIndex(groupColumns[L])
  if column not visible or mergeGroupedCells === false:
    continue
  
  segmentStart = 0
  currentKey = items[0]._groupPath[L]
  
  for i from 1 to items.length:
    if i === items.length OR items[i]._groupPath[L] !== currentKey:
      // Close segment [segmentStart, i)
      segmentLength = i - segmentStart
      items[segmentStart]._rowSpan[visibleColIndex] = segmentLength
      for j from segmentStart+1 to i-1:
        items[j]._rowSpanSkip[visibleColIndex] = true
      
      // Start new segment
      if i < items.length:
        segmentStart = i
        currentKey = items[i]._groupPath[L]
```

### 3.3 Metadata Schema

```ts
interface InlineMergedRowMetadata {
  _groupPath: any[];                    // [countryValue, genderValue, ...]
  _rowSpan: Record<number, number>;     // { visibleColIndex: spanCount }
  _rowSpanSkip: Record<number, boolean>; // { visibleColIndex: true if cell should not render }
}
```

---

## 4. Rendering: RowSpan-Aware Cell Generation

**Location:** `RowsView` extender for DataGrid (`packages/devextreme/js/__internal/grids/grid_core/views/m_rows_view.ts`)

### 4.1 Override `_renderCells($row, options)`

For `row.rowType === 'data'` when `displayMode === 'inlineMerged'`:

```ts
for each visibleColumn at index V:
  if row._rowSpanSkip?.[V] === true:
    // Skip rendering this <td> — cell from previous row spans into this row
    continue
  
  cellOptions = _getCellOptions(options, V)
  if row._rowSpan?.[V] > 1:
    cellOptions.rowspan = row._rowSpan[V]
  
  _renderCell($row, cellOptions)
```

**Key behavior:**

- Rows inside a rowSpan segment have **fewer `<td>` elements** than columns
- The first row of each segment has the merged `<td>` with `rowSpan` attribute

### 4.2 Update `_getCellOptions(options)`

Pass rowspan value to cell rendering:

```ts
if (row._rowSpan?.[visibleIndex]) {
  options.rowspan = row._rowSpan[visibleIndex];
}
```

---

## 5. DOM: Apply rowSpan Attribute

**Location:** `ColumnsView._createCell(options)` (`packages/devextreme/js/__internal/grids/grid_core/views/m_columns_view.ts`)

After existing `colSpan` logic:

```ts
if (options.rowspan > 1) {
  $cell.attr('rowSpan', options.rowspan);
}
```

### 5.1 Accessibility Considerations

- Verify `aria-colindex` uses logical column index, not DOM cell index
- Rows with skipped cells still report correct column positions
- Consider adding `aria-rowspan` for screen readers

---

## 6. Isolation: Preserve Classic Grouping

When `displayMode !== 'inlineMerged'` (default):

| Component | Behavior |
|-----------|----------|
| `_beforeProcessItems` | Calls `_processGroupItems()` as today |
| Group rows | `rowType: 'group'` items created with expand/collapse |
| `_rowClick` handler | Expands/collapses groups |
| RowSpan metadata | Not created; ignored if present |

**No changes** to existing group rendering, group row templates, or expand/collapse logic.

---

## 7. Edge Cases and Constraints

### 7.1 Initial Scope Limitations

| Scenario | Handling |
|----------|----------|
| `scrolling.mode: 'virtual'` | Disable `inlineMerged` or limit rowSpan to visible viewport rows |
| `scrolling.mode: 'infinite'` | Same as virtual |
| Remote grouping with paging | Treat page boundaries as segment terminators; no cross-page spanning |
| `isContinuation` groups | Treat as segment boundary; do not merge with previous page |
| Fixed columns | Requires special handling — rowSpan must be synchronized across fixed and scrollable table bodies |

### 7.2 Sorting and Filtering Interaction

- **Sorting:** Group columns are sorted by group order first, then by any additional `sortOrder` within groups
- **Filtering:** RowSpan recomputed after filter; segment sizes reflect filtered data
- **Column reordering:** Grouped columns should remain in group-order priority (leftmost = outermost group)

### 7.3 Editing and Selection

- **Row editing:** When editing a row inside a rowSpan, only that row's non-grouped cells become editable
- **Cell editing on merged cell:** Consider treating the merged cell as read-only or requiring `mergeGroupedCells: false`
- **Row selection:** Clicking the merged cell should select all rows in the span, or select only the first row (configurable)

---

## 8. API Summary

```ts
// Grid configuration
{
  dataSource: [...],
  grouping: {
    displayMode: 'inlineMerged'  // NEW
  },
  columns: [
    { dataField: 'Country', groupIndex: 0, showWhenGrouped: true },
    { dataField: 'Gender', groupIndex: 1, showWhenGrouped: true },
    { dataField: 'Name' },
    { dataField: 'Age' }
  ]
}
```

**Result:**

- Data rows render as flat list
- Country column cells row-span all people from that country
- Gender column cells row-span all people of that gender within each country
- Name and Age render per-row as normal

---

## 9. Implementation Priority

| Phase | Scope |
|-------|-------|
| Phase 1 | Single-level grouping with rowSpan; standard scrolling only |
| Phase 2 | Multi-level grouping with nested rowSpan |
| Phase 3 | Virtual scrolling support |
| Phase 4 | Fixed columns support |
| Phase 5 | Editing and selection refinements |

---

## 10. Open Questions

1. **Trigger mechanism:** Solely via `grouping.displayMode`, via per-column `mergeGroupedCells`, or a combination?
2. **Selection behavior:** Should clicking a merged cell select all spanned rows or just the first?
3. **Summary placement:** Where to render group summaries — in the merged cell, in a footer row, or both?
