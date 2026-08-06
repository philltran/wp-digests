# #80413: DataViewsPicker: Add Shift+Click range selection to the `picker-table`, `picker-grid`, and `picker-activity` layouts

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ntsekouras
- **Labels:** `[Type] Enhancement`, `[Feature] DataViews`, `[Package] DataViews`
- **Merged:** [`8b2bcdd`](https://github.com/WordPress/gutenberg/commit/8b2bcdd53676f0e1ebd98696db19cb4a877e05af)
- **Discussion:** [#80413](https://github.com/WordPress/gutenberg/pull/80413) · 4 comments · 1 reactions
- **Usefulness:** 4/5

## Summary

Adds Shift+Click range selection to the `picker-table`, `picker-grid`, and `picker-activity` layouts in the DataViews package. Previously, this gesture was only available in the standard `table` and `grid` layouts. The change centralizes selection logic in the `useSelectionProps` hook, which now accepts a `pickerMultiselect` flag to handle picker-specific behavior where every rendered item is selectable and plain clicks toggle selection.

## Impact

- **Block & plugin developers:** No breaking changes or public API removals. Picker layouts now natively support `Shift+Click` to select contiguous ranges, matching standard DataViews behavior.
- **DataViews consumers:** Single-select pickers ignore modifier gestures, while multi-select pickers extend the selection range from the last clicked anchor. Existing implementations require no migration unless they were manually overriding selection handlers.
- **No action required** for standard usage.

## Technical details

The diff refactors `packages/dataviews/src/components/dataviews-layouts/utils/use-selection-props.ts` to accept an optional `pickerMultiselect` parameter. When present, the hook treats every item in `data` as selectable (bypassing the `hasAPossibleBulkAction` filter used by other layouts) and returns an additional `onClick` handler in the `SelectionProps` interface. This handler toggles the item in the `selection` array and calls the new `anchorTo` helper to set the range anchor. The `picker-table`, `picker-grid`, and `picker-activity` components now call `useSelectionProps` with `pickerMultiselect: isMultiselect`, destructure `getSelectionProps`, and spread the resulting props onto their item containers instead of implementing inline click handlers. The `onClickCapture` handler in the hook now gates range extension on `hasRangeGesture` (which evaluates to `false` for single-select pickers), and `onMouseDown` prevents native text selection during `Shift+Click`.

## Contribution

Opened and merged by @ntsekouras, with co-authors @jasmussen and @jorgefilipecosta. The implementation builds on the `useSelectionProps` hook introduced in PR #80046. During review, @oandregal suggested making the hook fully layout-agnostic and opened a follow-up PR (#80677) to address that broader refactor. The author justified the `pickerMultiselect` prop by noting that picker layouts already declare their selection mode explicitly, making it a cleaner signal than deriving it from bulk action eligibility.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
