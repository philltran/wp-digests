# #83281: DataViews: Let a picker compose its footer from the page select and its actions

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jorgefilipecosta
- **Labels:** `[Type] Enhancement`, `[Package] DataViews`
- **Merged:** [`1288941`](https://github.com/WordPress/gutenberg/commit/12889415f1fa11f693c74f401ede850cbc202db8)
- **Discussion:** [#83281](https://github.com/WordPress/gutenberg/pull/83281) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The DataViewsPicker footer is now composable: `DataViewsPicker.Footer` and `DataViewsPicker.Pagination` accept `children` that replace their default contents, and three new sub-components — `DataViewsPicker.PageSelect`, `DataViewsPicker.PageNavigation`, and `DataViewsPicker.Actions` — expose the individual parts so consumers with limited UI space can pick which elements to render. Each part also accepts a `className`, letting the consumer's stylesheet control layout rather than the picker's. As a secondary change, the previously-unsupported `isEligible` action callback now works in picker footers, disabling the button when no selected item is eligible and passing only eligible items to the callback.

## Impact

- **Plugin & theme developers using DataViewsPicker:** New public sub-components (`DataViewsPicker.PageSelect`, `DataViewsPicker.PageNavigation`, `DataViewsPicker.Actions`) and `children`/`className` props on `DataViewsPicker.Footer` and `DataViewsPicker.Pagination`. No breaking changes — existing pickers render identically when no `children` are passed.
- **Picker action authors:** `isEligible` is now honored. If you previously relied on it being ignored (documented as unsupported), your action button will now be disabled when no selected item passes the check, and the callback receives a filtered array. Audit any picker actions that define `isEligible`.
- **Global Styles / Site Editor:** This PR is the DataViews prerequisite for #80856, which switches the Global Styles revisions screen to the `pickerActivity` layout. The `pickerActivity` layout type is now documented as supported for `DataViewsPicker`.
- **No action required** for existing pickers that do not pass `children` or define `isEligible`.

## Technical details

In `packages/dataviews/src/components/dataviews-pagination/index.tsx`, the monolithic `DataViewsPagination` was split into three exports:

- `DataViewsPageSelect({ className })` — the "Page X of Y" `SelectControl` dropdown.
- `DataViewsPageNavigation({ className })` — the previous/next `Button` pair.
- `DataViewsPagination({ children, className })` — the wrapper `Stack`. When `children` is provided it renders them in place of the default `<DataViewsPageSelect />` + `<DataViewsPageNavigation />` pair.

In `packages/dataviews/src/components/dataviews-picker-footer/index.tsx`:

- `PickerActions` (previously a private function) is now exported and accepts `{ className }`.
- `DataViewsPickerBulkActionToolbar` gains a `className` prop.
- `DataViewsPickerFooter({ children, className })` renders `children` in place of the default `<PickerBulkSelectionInfo />` + `<DataViewsPagination />` + `<PickerActions />` trio. The early-return guard (`! actions.length && ! hasPagination`) now only applies when `children` is not provided.
- `ActionButtons` now destructures `isEligible` from each action, computes `eligibleItems = isEligible ? items.filter(item => isEligible(item)) : items`, disables the button when `eligibleItems.length === 0`, and calls `callback(eligibleItems, { registry })` instead of `callback(items, { registry })`.

In `packages/dataviews/src/dataviews-picker/index.tsx`, three new properties are attached to the `DataViewsPicker` component object:

```ts
DataViewsPickerSubComponents.Actions = PickerActions;
DataViewsPickerSubComponents.PageNavigation = DataViewsPageNavigation;
DataViewsPickerSubComponents.PageSelect = DataViewsPageSelect;
```

Usage pattern (new):

```jsx
<DataViewsPicker.Footer>
  <DataViewsPicker.Pagination>
    <DataViewsPicker.PageSelect />
  </DataViewsPicker.Pagination>
  <DataViewsPicker.Actions className="my-screen__actions" />
</DataViewsPicker.Footer>
```

The README and CHANGELOG are updated to document `pickerActivity` as a supported `DataViewsPicker` layout type and to replace the "`isEligible` callback for actions is unsupported" note with the new behavior description. A new Storybook story (`free-composition.tsx`) demonstrates the composed footer with a `pagination` control that swaps between `page-select` and `page-navigation`.

## Contribution

Opened by @jorgefilipecosta as an extraction from #80856 at @ntsekouras's request, so that the Global Styles revisions screen PR could rebase on a smaller, independently mergeable DataViews change. @ntsekouras reviewed and asked for a free-composition Storybook story to be added; the author added it in response. The PR carries an AI-assistance disclosure for implementation and description drafting. Merged as commit `1288941` with co-author credit to both @jorgefilipecosta and @ntsekouras.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
