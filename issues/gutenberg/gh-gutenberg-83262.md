# #83262: Columns: Remove the column count slider

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Enhancement`, `[Package] Block library`, `[Block] Columns`
- **Merged:** [`d6fc7c6`](https://github.com/WordPress/gutenberg/commit/d6fc7c67e09eb439cc1aa60d19b91fff1ce2f839)
- **Discussion:** [#83262](https://github.com/WordPress/gutenberg/pull/83262) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The Columns block settings panel no longer includes a `RangeControl` slider for adjusting column count, nor the warning `Notice` that appeared when the count exceeded six. Columns are now added and removed exclusively through the standard block insert (plus button in the toolbar) and delete (Options menu) actions. The entire `packages/block-library/src/columns/utils.js` module—containing the width-redistribution helpers the slider depended on—and its test file are deleted.

## Impact

- **Site owners / content editors:** The slider in the Columns block settings is gone. To add a column, select a column and press the plus button next to the "Columns" parent button in the block toolbar. To remove one, select the column and use the Options menu. The "Stack on mobile" toggle remains the only setting in the panel. The six-column warning notice is also removed; there is no longer a UI-imposed cap on column count.
- **Plugin & theme developers:** No public API change. The deleted `utils.js` (`toWidthPrecision`, `getEffectiveColumnWidth`, `getTotalColumnsWidth`, `getColumnWidths`, `getRedistributedColumnWidths`, `hasExplicitPercentColumnWidths`, `getMappedColumnWidths`, `getWidths`) was an internal module under `packages/block-library/src/columns/` and was not exported from the `@wordpress/block-library` package entry point. No migration or code change is required.
- **No action required** for most developers. If a plugin or theme was reaching into the internal `src/columns/utils.js` path directly (non-standard), those imports will break.

## Technical details

In `packages/block-library/src/columns/edit.jsx`, the `ColumnInspectorControls` component is reduced to rendering only the "Stack on mobile" `ToolsPanelItem`. The removed code included:

- A `useSelect` call computing `count`, `canInsertColumnBlock`, and `minCount` from `blockEditorStore`.
- An `updateColumns(previousColumns, newColumns)` function that called `replaceInnerBlocks` with redistributed widths via `getRedistributedColumnWidths` / `getMappedColumnWidths` when explicit percent widths were present, or simply appended/sliced `core/column` blocks otherwise.
- A `RangeControl` (min `Math.max(1, minCount)`, max `Math.max(6, count)`) and a `Notice` (status `"warning"`) rendered inside a `VStack`.

The `clientId` prop is no longer passed to `ColumnInspectorControls` since the component no longer queries the block store for inner-block state.

The entire `packages/block-library/src/columns/utils.js` file (176 lines) and `packages/block-library/src/columns/test/utils.js` (337 lines) are deleted. The bundle size change is −621 B in `build/scripts/block-library/index.min.js`.

Before (slider present):
```jsx
<RangeControl
  label={ __( 'Columns' ) }
  value={ count }
  onChange={ ( value ) =>
    updateColumns( count, Math.max( minCount, value ) )
  }
  min={ Math.max( 1, minCount ) }
  max={ Math.max( 6, count ) }
/>
```

After: the `RangeControl` and surrounding `VStack`/`Notice` are absent; only the `ToolsPanelItem` for "Stack on mobile" remains.

## Contribution

Opened by @Mamaduka, closing long-standing issues #9009 and #10791. @jasmussen pushed back, arguing the slider was ergonomic and questioning whether the real problem was undo/revision-history discoverability rather than the control itself, and raised the upcoming Table block as a parallel case. @Mamaduka countered that the slider was inconsistent (capped at 6 while appenders allowed more; enabled rapid multi-column removal in one gesture) and that history handling had improved substantially since the original issue was filed. @talldan noted the Table block's add/remove model would differ (select specific rows/columns to delete) and did not see a parent-level slider as a common table pattern. After @Mamaduka asked whether to land it with no further strong objections, the PR was merged. The work was AI-assisted (Claude).

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
