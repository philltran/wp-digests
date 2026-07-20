# #80046: DataViews: Add shift-click range selection

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ntsekouras
- **Labels:** `[Type] Enhancement`, `Backport to Gutenberg RC`, `[Feature] DataViews`, `Backported to WP Core`, `[Package] DataViews`
- **Merged:** [`2eca416`](https://github.com/WordPress/gutenberg/commit/2eca416d45bfba5920b8e70b8759bc4e570ef48c)
- **Discussion:** [#80046](https://github.com/WordPress/gutenberg/pull/80046) · 21 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

DataViews now supports Shift+Click range selection across both table and grid layouts. This enhancement introduces a shared `useSelectionProps` hook that manages selection state, anchor tracking, and range calculation, aligning the interaction model with standard file managers and spreadsheets. It also standardizes Ctrl/Cmd+Click behavior to consistently trigger selection rather than allowing event propagation in the table layout.

## Impact

- **Plugin & theme developers using DataViews:** No immediate code changes required if you rely on default table/grid layouts. The selection behavior is now more predictable and consistent across layouts.
- **Developers building custom DataViews layouts:** You can opt into the new `useSelectionProps` hook to get Shift+Click and Ctrl/Cmd+Click range selection without reinventing the logic.
- **Site owners & editors:** Experience a more familiar multi-selection workflow in the Site Editor (Pages, Templates, etc.) and Media Library. Ranges now extend predictably from the last directly clicked item, and reversing direction redefines the range from that anchor.
- **Behavior change for table layouts:** The previous table layout allowed Ctrl/Cmd+Click to propagate to links. This PR aligns both layouts to handle modifier clicks exclusively for selection, which may affect custom table implementations relying on link navigation during modifier clicks.

## Technical details

The diff introduces `packages/dataviews/src/components/dataviews-layouts/utils/use-selection-props.ts`, which exports `getRangeSelection` and `getClosestSelectedId`. The `useSelectionProps` hook accepts `data`, `actions`, `getItemId`, `selection`, and `onChangeSelection`, returning a `getSelectionProps` function that attaches `onMouseDown` and `onClickCapture` handlers to layout items.

- In `table/index.tsx` and `grid/index.tsx`, the hook is instantiated at the view level. For grouped data, `orderedData` flattens group values to ensure ranges span across groups with a single anchor.
- The `TableRow` and `CompositeGrid` components now spread `getSelectionProps( id )` onto their root elements, replacing inline modifier-key logic.
- The table layout's previous `isTouchDeviceRef` and inline `Ctrl/Cmd+Click` handling were removed. Firefox's cell-selection interference is still prevented via `event.preventDefault()` on `onMouseDown`.
- `dataviews-bulk-actions/index.tsx` extracts `hasAPossibleBulkAction` to reduce `useMemo` nesting and improve readability.
- Selection state now tracks an `anchorId` and `lastTargetId` to handle direction reversal and deselection edge cases, matching the behavior described in the PR notes.

## Contribution

Opened and merged by @ntsekouras, with co-authors @oandregal, @jasmussen, @fcoveram, @talldan, and @simison. The PR addressed reviewer feedback from @oandregal regarding anchor persistence when reversing selection direction and behavior when all items are deselected. The final implementation keeps the anchor fixed across consecutive shift-clicks and aligns both table and grid layouts to handle modifier clicks consistently for selection. The change was backported to the Gutenberg RC and WordPress Core.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
