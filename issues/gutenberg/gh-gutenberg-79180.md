# #79180: [DataViewsPicker]: `DataViewsPicker.BulkActionToolbar` now renders only the bulk-selection info and action buttons

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ntsekouras
- **Labels:** `[Type] Enhancement`, `[Package] Media Utils`, `[Package] DataViews`
- **Merged:** [`c558d9d`](https://github.com/WordPress/gutenberg/commit/c558d9de2c1493aa7a140112f5682645e9a1f7f5)
- **Discussion:** [#79180](https://github.com/WordPress/gutenberg/pull/79180) · 2 comments · 1 reactions
- **Usefulness:** 4/5

## Summary

DataViewsPicker.BulkActionToolbar has been refactored to render only bulk-selection info and action buttons, removing pagination from its output. The complete footer, including pagination, is now exposed as a separate DataViewsPicker.Footer component. This alignment with DataViews improves component composition and prepares the picker for revisions screen updates.

## Impact

- **Plugin & theme developers**: This is a breaking change. If you were rendering DataViewsPicker.BulkActionToolbar and relied on it to display pagination, you must migrate to DataViewsPicker.Footer.
- **Media Utils consumers**: The MediaUploadModal component was updated to use DataViewsPicker.Footer, so existing media upload flows remain unchanged.
- **No action required** if you were not directly rendering DataViewsPicker.BulkActionToolbar or DataViewsPicker.Footer.

## Technical details

The change lives in `packages/dataviews/src/components/dataviews-picker-footer/index.tsx` and `packages/dataviews/src/dataviews-picker/index.tsx`. Previously, `DataViewsPicker.BulkActionToolbar` pointed to the monolithic `DataViewsPickerFooter` component, which bundled selection info, pagination, and action buttons. It now points to the newly exported `DataViewsPickerBulkActionToolbar`, which composes `PickerBulkSelectionInfo` and `PickerActions`. Pagination was moved out of this toolbar and into `DataViewsPickerFooter`, which now renders `PickerBulkSelectionInfo`, `DataViewsPagination`, and `PickerActions`.

The PR also fixes a logic bug in `useIsMultiselectPicker` by adding an `!! actions?.length` guard before checking `action.supportsBulk`, preventing incorrect `aria` props when no actions are defined.

Before/after usage pattern:
```jsx
// Before
<DataViewsPicker.BulkActionToolbar /> // rendered pagination + actions

// After
<DataViewsPicker.BulkActionToolbar /> // renders only selection info + actions
<DataViewsPicker.Footer />            // renders selection info + pagination + actions
```

## Contribution

Extracted from a larger refactor in #77333, the change was driven by the need to improve footer component organization for upcoming revisions screen updates. The author separated pagination from bulk actions to enable free composition, and the PR landed after a quick review with no major design debates.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
