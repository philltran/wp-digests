# #79134: Widget Primitives: extract into `@wordpress/widget-primitives`

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `npm Packages`, `[Feature] Dashboard`
- **Merged:** [`e733a66`](https://github.com/WordPress/gutenberg/commit/e733a66647889abbafeb996f5285120031a2c366)
- **Discussion:** [#79134](https://github.com/WordPress/gutenberg/pull/79134) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The dashboard widget toolkit has been extracted from the internal `routes/dashboard/widget-primitives/` directory into a standalone, publishable npm package: `@wordpress/widget-primitives`. This change establishes a formal, host-agnostic contract for dashboard widgets, exposing the core types, a discovery hook, and a render component without altering existing behavior. It matters because it decouples the widget contract from the WordPress admin route, enabling external hosts and plugin authors to consume the widget system independently.

## Impact

- **Plugin & theme developers / host authors:** No immediate action required. The package is experimental (`0.1.0-prerelease`) and currently consumed only behind the `gutenberg-dashboard-widgets` experiment.
- **Internal Gutenberg contributors:** No breaking changes or deprecations, but internal import paths have shifted from `routes/dashboard/widget-primitives/` to `@wordpress/widget-primitives`. Developers updating the dashboard route or building new widget hosts should import from the new package.
- **Site owners:** No action required. The change is internal to the Gutenberg build pipeline and does not affect frontend or admin behavior outside the experiment.

## Technical details

The diff performs a structural extraction rather than a behavioral rewrite. Source files were moved from `routes/dashboard/widget-primitives/` to `packages/widget-primitives/src/` with zero logic changes. A new `package.json` configures the package for public publishing (`publishConfig.access: public`), sets `wpScript: false`, and declares peer dependencies on `react` and `@types/react`. The package exposes:
- Contract types: `WidgetType`, `WidgetName`, `WidgetIcon`, `WidgetRenderProps`, `ResolveWidgetModule`
- Discovery hook: `useWidgetTypes()` (reads the `widgetModule` core-data entity backed by `/wp/v2/widget-modules`)
- Host entry point: `<WidgetRender>` (resolves `renderModule` via a host-provided `ResolveWidgetModule` and mounts the component with `attributes`/`setAttributes`)
The dashboard route now consumes the package via a `file:` workspace dependency, and its `tsconfig.json` reference to the old internal path was removed. Architecture documentation was added at `docs/explanations/architecture/dashboard-widgets.md` to formalize the authoring convention, build pipeline, server registry (`WP_Widget_Type_Registry`), and host contract.

## Contribution

Opened and merged by @retrofox with co-authorship from @chihsuan; the PR record carries no substantive design discussion or alternative approaches, reflecting a straightforward extraction of an existing internal boundary into a publishable package.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
