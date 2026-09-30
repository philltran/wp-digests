# #82601: DataViews: Skip table columns for field ids without a field definition

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @oandregal
- **Labels:** `[Type] Bug`, `[Feature] DataViews`, `[Package] DataViews`
- **Merged:** [`233bec4`](https://github.com/WordPress/gutenberg/commit/233bec4969a0d008c45e0e5b1682b288d5baca9a)
- **Discussion:** [#82601](https://github.com/WordPress/gutenberg/pull/82601) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `table` and `pickerTable` layouts of DataViews now skip any id in `view.fields` that has no matching field definition, instead of rendering an empty `<col>`, `<th>` and `<td>` for it. The grid, picker grid and list layouts already dropped such ids, so this brings the table layouts in line. The bug surfaced because the `wp_template` default view served by `GET /wp/v2/view-config` lists `active`, `slug` and `theme`, which neither site editor registers as fields.

## Impact

- **Plugin & theme developers using DataViews / DataViewsPicker:** Views that reference unregistered field ids (stale persisted views, or server-provided defaults naming fields the screen never registered) no longer produce phantom columns. Empty headers, extra padding width, inflated grouped-row `colSpan` and empty keyboard tab stops go away.
- **Behavior note:** The column header menu now computes move/insert/hide against the rendered columns. Skipped ids are dropped from `view.fields` the next time the menu changes the view (per the CHANGELOG entry).
- **Custom code that relied on phantom columns:** Unlikely, but any CSS or tests targeting the empty `dataviews-view-table__col-<id>` columns for unregistered ids will no longer match.
- No migration or code changes required.

## Technical details

A new helper, `getTableColumns( view, fields )`, lives in `packages/dataviews/src/components/dataviews-layouts/utils/get-table-columns.ts`. It returns `( view.fields ?? [] )` filtered to ids for which some normalized field has a matching `id`, preserving view order.

It replaces `view.fields ?? []` in four places:
- `table/index.tsx`: both `TableRow` and `ViewTable`
- `picker-table/index.tsx`: both `TableRow` and `ViewPickerTable`

The header row, item rows and `colSpan` therefore stay in sync.

`table/column-header-menu.tsx` also changed. `visibleFieldIds` and `index` were previously derived from raw `view.fields` at the top of the component. They are now computed after the early return, from `getTableColumns( view, fields )`. Move left/right and insert operations therefore use indexes into the rendered columns, so a skipped id can't offset them.

```ts
// before
const columns = view.fields ?? [];
// after
const columns = getTableColumns( view, fields );
```

Tests added:
- Unknown id renders no header or cell, in both `dataviews.jsdom.test.tsx` and `dataviews-picker.jsdom.test.tsx`.
- Moving a column right skips past an unknown id. With `[ 'title', 'missing', 'order' ]`, the result is `[ 'order', 'title' ]`.
- The last rendered column has "Move right" disabled even when a skipped id follows it.

No hooks, REST schema or DB changes. A CHANGELOG bug-fix entry was added for `@wordpress/dataviews`.

## Contribution

Opened by @oandregal after spotting the phantom `active`/`slug`/`theme` columns while reviewing #82456. The PR was written with Claude Code and reviewed and tested by the author. The discussion is minimal: @manzoorwanijk briefly suspected this PR of breaking trunk unit tests, then traced the failure to an unrelated `wordpress-develop` commit.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
