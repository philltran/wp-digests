# #78941: DataViewsPicker: Add a new `pickerActivity` layout

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ntsekouras
- **Labels:** `[Type] Enhancement`, `[Feature] DataViews`, `[Package] DataViews`
- **Merged:** [`e746a6c`](https://github.com/WordPress/gutenberg/commit/e746a6c74a558a46b84cd1373083762cab12e808)
- **Discussion:** [#78941](https://github.com/WordPress/gutenberg/pull/78941) · 6 comments · 0 reactions

## Summary

The `DataViewsPicker` component now supports a new `pickerActivity` layout, extending the existing DataViews activity layout specifically for picker contexts. This additive feature introduces an alternative presentation mode optimized for selecting items based on status or changes, directly supporting upcoming Post Revisions UI implementations and Global Styles revision workflows.

## Impact

['- **Block & Plugin Developers**: No breaking changes; this is a purely additive component variant.', '- **Editor/Platform Teams**: Enables adopting a more compact, activity-centric picker layout for features like Post Revisions and Global Styles.', '- **Action Required**: None required for existing integrations. To utilize the new layout, pass `layout="pickerActivity"` to the `<DataViewsPicker>` component.']

## Technical details

["- Registers a `pickerActivity` layout variant within the `@wordpress/dataviews` package's `DataViewsPicker` component.", '- Mirrors the structural and styling conventions of the standard DataViews activity layout but adapts spacing, typography, and interaction states for picker contexts.', '- The implementation modifies internal rendering logic for the picker wrapper; no REST schema changes, database modifications, or public API deprecations are introduced.']

## Contribution

Merged in commit `e746a6c`, this enhancement was extracted from PR #77333. @ntsekouras authored the patch, with co-authors @t-hamano and @jameskoster joining the review process. The team separated this layout variant to decouple picker-specific rendering from the core DataViews suite, preparing the foundation for the upcoming post revisions screen and Global Styles revision interface.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
