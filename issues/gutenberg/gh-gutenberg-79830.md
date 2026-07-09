# #79830: Widgets: add a declarative `help` metadata field, surfaced as a header infotip

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Package] wp-build`, `[Package] Widget primitives`, `[Package] Widget Dashboard`
- **Merged:** [`af5391e`](https://github.com/WordPress/gutenberg/commit/af5391e7f5eb4661f945b1cd181447b2b17fc2f8)
- **Discussion:** [#79830](https://github.com/WordPress/gutenberg/pull/79830) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Adds an optional `help` metadata field to the experimental widget system, enabling developers to declare short contextual notes (`content`) and supplementary links (`links`). The field flows from `widget.json` through the build manifest and PHP registry, is sanitized and translated server-side, and is exposed via the `/wp/v2/widget-modules` REST controller. On the frontend, the dashboard header surfaces it as a clickable infotip popover, filling a gap where widgets previously only offered `title` and `description`.

## Impact

- **Widget authors & UI developers**: Can opt-in by adding `help` to `widget.json`; no breaking changes or migration required.
- **Dashboard consumers**: Automatically receive the structured help payload in REST responses; rendering is handled internally by `WidgetHeader`.
- **Strictly additive and backwards-compatible**: Existing widgets without `help` render unaffected, and the REST schema gracefully returns `null` when absent.

## Technical details

- **Schema & Registry**: `WP_Widget_Type::$help` is added as a nullable property. The `/wp/v2/widget-modules` REST controller (`class-wp-rest-widget-modules-controller.php`) exposes it via a typed `object|null` schema containing `content` (string) and `links` (array of `{label, href}` objects).
- **PHP Pipeline**: `widget-i18n.json` defines the i18n structure. `gutenberg_translate_widget_metadata()` processes `help` through `translate_settings_using_i18n_schema()`. A new `gutenberg_sanitize_widget_help()` function constrains `content` to `<em>`/`<strong>` via `wp_kses` and drops malformed link entries before hydrating the registry in `gutenberg_register_widget_types()`.
- **JavaScript/Dashboard**: `packages/widget-primitives/src/types.ts` adds `WidgetHelp` and `help?: WidgetHelp` to `WidgetTypeMetadata`. `useWidgetTypes()` overlays server-resolved `help` from `WidgetModuleRecord`. `WidgetHeader` conditionally renders `WidgetInfotip`, which implements a `<Popover.Root>` that parses the sanitized content and maps declared links.
- **Build**: `packages/wp-build/lib/build.mjs` serializes `help` from `widget.json` into the generated PHP registry payload.

## Contribution

Opened and merged as PR #79830 by @retrofox (co-authored by @chihsuan). The implementation follows a declarative data-only pattern established in recent dashboard work (#77629, #79746), keeping widgets responsible for declaring `content`/`links` while the host dictates rendering. Follow-up tasks remain open for surfacing `help` in the widget picker and evaluating primary CTA link variants.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
