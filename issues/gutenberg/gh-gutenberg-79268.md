# #79268: Widget Dashboard: extract into `@wordpress/widget-dashboard`

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Package] Private APIs`, `[Feature] Dashboard`
- **Merged:** [`cbf74db`](https://github.com/WordPress/gutenberg/commit/cbf74db259dea379181f2d198190f76cfeaea70a)
- **Discussion:** [#79268](https://github.com/WordPress/gutenberg/pull/79268) · 3 comments · 1 reactions
- **Usefulness:** 3/5

## Summary

The experimental dashboard widget engine has been extracted from `routes/dashboard/widget-dashboard/` into a standalone, publishable package at `packages/widget-dashboard/`, exposed as `@wordpress/widget-dashboard` (`0.1.0-prerelease`). This change decouples the rendering logic from the admin route, allowing external consumers to build custom dashboards using the same drag-to-reorder, resize, and modal inserter capabilities. The package ships as an experimental API dependent on core-private APIs, meaning it is intended for WordPress core consumption rather than standalone plugin use at this stage.

## Impact

- **Plugin & theme developers:** No action required. The package is marked experimental and depends on core-private APIs, making it unsafe for standalone plugin consumption.
- **Core contributors & host implementers:** The dashboard route now imports from `@wordpress/widget-dashboard` instead of housing the engine inline. Hosts must pass `widgetTypes`, `layout`, and `onLayoutChange` to the `WidgetDashboard` component.
- **Platform & hosting teams:** No configuration changes needed. The package is registered in `packages/private-apis/src/implementation.ts` and will be bundled with core.
- **Breaking changes:** None for external consumers. Internal component names were refactored (e.g., `WidgetChrome`, `Commands`), but these were not public APIs.

## Technical details

The diff moves the engine from `routes/dashboard/widget-dashboard/` to `packages/widget-dashboard/src/`, establishing a composite `tsconfig`, `README`, and `CHANGELOG`. It registers the package in `packages/private-apis/src/implementation.ts` under `CORE_MODULES_USING_PRIVATE_APIS`. The main export is the `WidgetDashboard` compound component, which accepts `layout`, `onLayoutChange`, `widgetTypes`, `editMode`, `onEditChange`, `resolveWidgetModule`, and `gridSettings`. Grid configuration utilities (`DEFAULT_GRID`, `normalizeGridSettings`, `ROW_HEIGHT_PRESETS`, `DEFAULT_ROW_HEIGHT`, `WIDGET_DASHBOARD_COLUMN_COUNT`) are exported for host-side persistence. The package declares `react` and `react-dom` as peer dependencies, moves `@types/react` to optional peer dependencies, and adds a local ambient type declaration for `@wordpress/commands` (`src/wordpress-commands.d.ts`) to satisfy strict type checking. The admin route now consumes the package, retaining only layout persistence and data wiring responsibilities.

## Contribution

Opened and merged by @retrofox with co-authors @manzoorwanijk and @chihsuan. The change consolidates the dashboard rendering engine into a reusable package to enable external dashboard implementations, following the established pattern of publishing internal engines under `./packages`. The discussion contains only automated size and test reports, with no recorded design debates or rejected alternatives.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
