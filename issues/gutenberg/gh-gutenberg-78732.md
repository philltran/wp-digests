# #78732: Dashboard: Opinionated grid columns with container breakpoints

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jameskoster
- **Labels:** `[Type] Enhancement`, `[Feature] Dashboard`
- **Merged:** [`8f4766b`](https://github.com/WordPress/gutenberg/commit/8f4766b84ae35498d41822ab47e58f0e66ce30fa)
- **Discussion:** [#78732](https://github.com/WordPress/gutenberg/pull/78732) · 5 comments · 0 reactions

## Summary

The WordPress Dashboard now enforces an opinionated, responsive grid layout that ignores previously persisted column settings. Instead of exposing UI controls for tile count and minimum width, the dashboard hard-codes a maximum of four columns on wide containers and automatically steps down to two columns at 600px–959px and one column below 600px based on the container's actual width. This change removes low-level grid configuration options that rarely improve usability and simplifies the layout settings drawer.

## Impact

- **Dashboard Users & Editors**: The *Columns*, *Adjust on narrow screens*, and *Minimum tile width* controls are permanently removed from the Layout settings drawer. Any previously saved `columns` or `minColumnWidth` user preferences are ignored at runtime.
- **Plugin & Theme Developers**: No public API changes affect standard widget rendering. Developers who programmatically read or manipulate dashboard layout state via internal preferences APIs will encounter a hardcoded column resolution; no code changes are required unless specifically leveraging the removed grid settings.

## Technical details

- Drops `Columns`, `Adjust on narrow screens`, and `Minimum tile width` from the dashboard Layout UI.
- Introduces `WIDGET_DASHBOARD_COLUMN_COUNT` (hardcoded to `4`) and a `resolveDashboardColumnCount()` function to compute layout at runtime.
- Implements `useDashboardContainerColumnCount`, which uses a `ResizeObserver` on the `.grid` wrapper to measure container width directly, decoupling tile steps from viewport media queries.
- Applies container-query alignment via `container-type: inline-size` and `container-name: widget-dashboard` on the grid wrapper element.
- Updates defaults in `WidgetDashboardProvider` and `useDashboardGridSettings` to stop passing `minColumnWidth` to `@wordpress/grid`, aligning with the new opinionated breakpoints (≥960px → 4 cols, 600–959px → 2 cols, <600px → 1 col).
- **Before**: Layout settings exposed adjustable grid parameters that persisted to user options and passed through to the grid engine.
- **After**: Grid column count is resolved dynamically via container measurement; persisted tile layout preferences are disregarded in favor of the hardcoded responsive steps.

## Contribution

Opened and merged by @jameskoster as a dashboard enhancement. The PR consolidates grid geometry into fixed container-based breakpoints, shifting layout control from user-adjustable settings to a deterministic JS measurement flow. Design rationale centered on reducing cognitive load for low-value configuration while ensuring consistent tile readability across varying dashboard panel widths.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
