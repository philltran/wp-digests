# #78821: Dashboard: Promote WidgetRender into widget-primitives

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Feature] Dashboard`
- **Merged:** [`943b672`](https://github.com/WordPress/gutenberg/commit/943b67221a88e50cda540f97137f7fa752c9dc6d)
- **Discussion:** [#78821](https://github.com/WordPress/gutenberg/pull/78821) · 3 comments · 0 reactions

## Summary

This is a pure refactor of the WordPress dashboard's widget rendering code in Gutenberg — no behavior change. It promotes the host-agnostic `<WidgetRender>` component out of the dashboard route and into the `widget-primitives` kit, so it becomes the single canonical entry point for mounting a dashboard widget. The dashboard's old `WidgetRender` is reduced to a thin adapter renamed `DashboardWidgetRender` that only supplies dashboard-specific concerns (context wiring and the per-instance `setAttributes` callback). The same pass narrows the `widget-primitives` public API and trims stale/speculative documentation.

## Impact

This affects only internal Gutenberg `routes/dashboard` code; there is no public WordPress API, REST schema, hook, or block.json change.

- **Site owners / end users:** No action required. Widgets, the widget picker (inserter), and attribute editing behave exactly as before.
- **Gutenberg/dashboard contributors:** If you build a host for dashboard widgets, import `<WidgetRender>` from `widget-primitives` rather than copying the dashboard wrapper. Suspense, error handling, and chrome remain host concerns and are *not* part of the primitive.
- **API consumers of `widget-primitives`:** Two symbols are removed from the package's public surface — `getLazyWidgetComponent` (now used only internally by the kit's own `WidgetRender`) and the `WidgetTypeMetadata` type (no external consumer). The package now exports `<WidgetRender>`, `useWidgetTypes()`, and the contract types `WidgetType`, `WidgetName`, `WidgetRenderProps`, and `ResolveWidgetModule`. Existing unit tests pass unchanged because they only consume the still-stable contract types.

## Technical details

Three structural moves plus a documentation trim:

1. **New host-agnostic component** at `routes/dashboard/widget-primitives/components/widget-render/`. `<WidgetRender>` exposes the standard render contract — `widgetType`, `attributes`, `setAttributes`, `resolveWidgetModule` — and internally consumes `getLazyWidgetComponent`.
2. **Dashboard adapter** at `routes/dashboard/widget-dashboard/components/widget-render/`, whose export is renamed from `WidgetRender` to `DashboardWidgetRender`. It now does only two things: bridge dashboard state via `useDashboardInternalContext()`, and build the per-instance `setAttributes` callback that dispatches attribute updates against the dashboard layout. Consumers `dashboard-widget-chrome` and `widget-picker` are updated to the new name.
3. **Narrowed public API:** `widget-primitives` stops exporting `getLazyWidgetComponent` and `WidgetTypeMetadata`.

Usage pattern for a host changes from importing the dashboard wrapper to importing the primitive directly:

```js
// Before — host-specific wrapper was the only mount point
import { WidgetRender } from '../widget-dashboard/components/widget-render';

// After — canonical primitive for any host
import { WidgetRender } from '../widget-primitives';
```

Documentation across `README.md`, `types.ts`, the hook, the tool, and the new component is corrected: broken symbol references (`WIDGET_NAME_REGEXP in registerWidgetType`, a `getWidgetTypes` resolver — neither exists) are replaced with the real mechanisms, `WP_Widget_Type_Registry::register()` and the `useWidgetTypes` hook. Speculative tooling-consumer lists, restatement-style JSDoc, and an out-of-date folder enumeration in the README are removed. The reported bundle size change is 0 B.

## Contribution

Opened and merged by **@retrofox**, with **@simison** credited as co-author. The PR is labeled `[Type] Enhancement` / `[Feature] Dashboard` and merged as commit `943b672`. Discussion was light (3 comments, largely the props-bot and bundle-size bot), so there is no recorded design debate or rejected alternative — the change is framed as straightforward groundwork to give future widget hosts (a sidebar, inserter, or digest area) a single canonical render entry point instead of copying the dashboard wrapper.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
