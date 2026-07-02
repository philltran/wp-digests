# #79701: Widgets: translate `title`, `description`, and `keywords` server-side

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Package] wp-build`, `[Package] Widget primitives`
- **Merged:** [`090f0f9`](https://github.com/WordPress/gutenberg/commit/090f0f9d34cc97ef8840511a0299ce242d2758da)
- **Discussion:** [#79701](https://github.com/WordPress/gutenberg/pull/79701) · 3 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

This change brings server-side localization of dashboard widget metadata to parity with block registration. The build pipeline now forwards `title`, `description`, and `keywords` from `widget.json` into the PHP registry, where a new helper function localizes them at registration using each widget's declared `textdomain`. The REST endpoint `/wp/v2/widget-modules` now returns these localized strings, allowing hosting environments to serve translated copy without relying on JavaScript i18n loading.

## Impact

- **Plugin & Theme Developers / Widget Hosts**: Existing widgets continue to function, but the build pattern has shifted. Static strings should reside in `widget.json` with a corresponding `textdomain`, replacing the previous duplicate JS-side `__()` calls in `widget.ts`. Host implementations should expect server-localized metadata to take precedence.
- **Headless & REST Consumers**: The `/wp/v2/widget-modules` schema is expanded to include localized `title`, `description`, and `keywords` fields. API consumers will now receive translated payloads instead of raw source strings.
- **No immediate action required** for standard dashboard widget users, as this is an internal consistency improvement that surfaces translated metadata automatically.

## Technical details

The merge introduces `lib/experimental/dashboard-widgets/widget-i18n.json` to map translatable fields to Gettext contexts. Inside `widget-types.php`, `gutenberg_register_widget_types()` now calls a new `gutenberg_translate_widget_metadata()` helper, which applies `translate_settings_using_i18n_schema()` against each widget's `textdomain` before hydrating the `WP_Widget_Type`. The REST controller (`class-wp-rest-widget-modules-controller.php`) conditionally injects these fields into responses via `rest_is_field_included()` and extends its schema to expose them as `string|null` or `string[]|null`. On the client, `packages/widget-primitives/src/hooks/use-widget-types.ts` updates the overlay fallback chain so that server-localized values from `record.title`, `record.description`, and `record.keywords` win over module defaults, falling back to `record.name`. The `WidgetModuleRecord` TypeScript interface is updated to accept these optional overrides.

## Contribution

Opened and merged by `@retrofox` (with co-authorship from `@chihsuan`) as commit `090f0f9d34cc97ef8840511a0299ce242d2758da`. The implementation followed the established i18n schema pattern used for blocks, consolidating translatable metadata into `widget.json` and shifting localization responsibility to the server-side registry during registration.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
