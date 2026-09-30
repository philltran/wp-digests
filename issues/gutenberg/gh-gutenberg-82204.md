# #82204: Widget Dashboard: make the column count a host decision

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Nikschavan
- **Labels:** `[Type] Enhancement`, `[Package] Widget Dashboard`
- **Merged:** [`e92adf6`](https://github.com/WordPress/gutenberg/commit/e92adf6d0cbe416859f3b1870aef24865b74bb9d)
- **Discussion:** [#82204](https://github.com/WordPress/gutenberg/pull/82204) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

`@wordpress/widget-dashboard` now honors `gridSettings.columns` as the wide-container column count instead of overwriting it with a hard-coded four. The value is floored at one with no upper limit, and `WIDGET_DASHBOARD_COLUMN_COUNT` (4) becomes only the fallback when the host sets nothing. The responsive steps scale from the chosen count (`count → min( 2, count ) → 1`). The wp-admin Dashboard (Beta) still renders four columns because the route now pins that value itself.

## Impact

**Plugin/theme/app developers embedding `WidgetDashboard`**
- `gridSettings.columns` now takes effect. Previously it was silently ignored. Hosts wanting 3 (e.g. Jetpack Stats) or 12 (a customizable grid) can now set it.
- Behavior change: if you already pass `columns` (e.g. from persisted payloads) and relied on it being ignored, the dashboard will now render that count. Non-finite or absent values still resolve to 4.
- There is no ceiling, so a large `columns` value renders as asked. Tile spans are stored per widget and don't scale with the count, so raising the count makes the same spans cover less of the surface.
- A CSS override was never viable because drag and resize read the same `columns` value; this is now the supported route.

**wp-admin Dashboard (Beta) users**
- No visible change. It stays 4 → 2 → 1. Preferences persisted by the removed Columns control (values 1–12 under `core/dashboard.dashboardGridSettings`) remain inert.

**Storybook / contributors**
- New **Widget Dashboard → Playground → Grid Settings** story with columns (1–12), model, and row height controls.

## Technical details

**New helper** in `utils/resolve-dashboard-column-count/resolve-dashboard-column-count.ts`:

```ts
export function resolveDashboardColumnCap( columns?: number ): number {
	if ( typeof columns !== 'number' || ! Number.isFinite( columns ) ) {
		return WIDGET_DASHBOARD_COLUMN_COUNT;
	}
	return Math.max( 1, Math.floor( columns ) );
}
```

**Changed behavior**
- `resolveGridSettings()` in `context/dashboard-context.tsx`: before, `columns: WIDGET_DASHBOARD_COLUMN_COUNT`; now `columns: resolveDashboardColumnCap( normalized.columns )`.
- `resolveDashboardColumnCount( containerWidth, maxColumns = WIDGET_DASHBOARD_COLUMN_COUNT )` returns `maxColumns` when width is `<= 0` or `>= 960`, `Math.min( 2, maxColumns )` for 600–959px, and 1 below 600px. The `min` keeps a cap of one from stepping up to two.
- `useDashboardContainerColumnCount( forwardedRef, maxColumns )` accepts the cap and includes it in its `useMemo` deps. `Widgets` passes `gridSettings.columns`.

**wp-admin pin**: per the PR description, `useDashboardGridSettings()` in the dashboard route now returns `columns: WIDGET_DASHBOARD_COLUMN_COUNT` after reading the stored preference. That file's diff was truncated from the provided input, so this is taken from the description and review discussion rather than verified in the diff.

**Types/docs**: JSDoc on `WIDGET_DASHBOARD_COLUMN_COUNT` and `BaseWidgetGridSettings.columns` now describes a default rather than a maximum; the README and CHANGELOG (Enhancements) are updated to match. Unit tests were added for `resolveDashboardColumnCap` (default, floor of 1, fractional flooring, NaN/Infinity, counts above 4) and for `resolveDashboardColumnCount` with a host cap. The PR description states 15 tests pass. No new hooks, REST routes, or DB changes.

## Contribution

The PR first proposed a cap on the column count, but @retrofox's review argued the ceiling was in the wrong layer: the package is a rendering engine that knows neither the surface width nor the application, so the four-column limit belongs to the wp-admin page. He also flagged that the old overwrite in `resolveGridSettings()` was what kept stale preferences from the removed Columns control inert, so the pin had to move into `useDashboardGridSettings()` rather than disappear. @Nikschavan adopted that direction (floor-only resolver, pin in the route, plus the Grid Settings story) and retitled the PR to match.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
