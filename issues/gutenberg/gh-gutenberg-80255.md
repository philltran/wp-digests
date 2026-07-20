# #80255: DataViews: Fix the `list` layout ignoring some settings when `groupBy` is set

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ntsekouras
- **Labels:** `[Type] Bug`, `Backport to Gutenberg RC`, `[Feature] DataViews`, `Backported to WP Core`, `[Package] DataViews`
- **Merged:** [`c58009e`](https://github.com/WordPress/gutenberg/commit/c58009e934a26d13cd5e59e263a8123c7e438571)
- **Discussion:** [#80255](https://github.com/WordPress/gutenberg/pull/80255) · 7 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `list` layout in the DataViews component now correctly applies density settings, loading states, and refreshing states when items are grouped via `groupBy`. Previously, these visual and accessibility states were only applied to the ungrouped rendering path, causing grouped lists to render with incorrect styling and missing `inert` or `is-refreshing` attributes. This fix aligns the grouped path with the shared layout logic used by other DataViews layouts.

## Impact

- **Plugin & theme developers using DataViews:** No code changes required. Grouped list views will now correctly reflect `density` settings and apply `is-refreshing` and `inert` attributes during data loading.
- **Admin UI users:** Grouped list views in the block editor will now behave consistently with ungrouped views, including proper keyboard navigation states and loading feedback.
- **No action required** for existing implementations; this is a transparent bug fix that restores expected behavior.

## Technical details

The change modifies `packages/dataviews/src/components/dataviews-layouts/list/index.tsx` in the `ViewList` component. Previously, the grouped rendering branch (`hasData && groupField && dataByGroup`) passed inline props to `<Composite>` and `<Stack>`, bypassing shared styling and state logic. The diff extracts `listClassName` (applying `has-{density}-density` and `is-refreshing` via `clsx`) and `compositeProps` (containing `ref`, `id`, `render`, `activeId`, `setActiveId`, and `inert`) into shared variables. Both the grouped and ungrouped branches now spread `{ ...compositeProps }` and use `listClassName` on the `<Stack>` wrapper. Unit tests in `packages/dataviews/src/dataviews/test/dataviews.tsx` verify that `.dataviews-view-list` receives the correct density class and that the `inert` attribute and `is-refreshing` class are applied correctly during loading states for both grouped and ungrouped configurations.

## Contribution

Opened by @ntsekouras and co-authored by @ciampo, the PR extracts a shared configuration object to unify the grouped and ungrouped rendering paths in the `list` layout, mirroring the approach used in the `grid` layout. @t-hamano reviewed and manually cherry-picked the commit to the `wp/7.1` branch for inclusion in the next release. The discussion focused on aligning the implementation with existing layout patterns rather than introducing new behavior.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
