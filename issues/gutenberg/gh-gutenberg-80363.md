# #80363: Dashboard Widgets: declarative actions in the widget schema

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Package] wp-build`, `[Package] Widget primitives`, `[Package] Widget Dashboard`
- **Merged:** [`9faa7de`](https://github.com/WordPress/gutenberg/commit/9faa7de501a61628a90b2fddc2219c8798adc5df)
- **Discussion:** [#80363](https://github.com/WordPress/gutenberg/pull/80363) · 6 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Adds an optional `actions` field to the experimental dashboard widget schema, allowing widget authors to declaratively specify navigation and download verbs. The dashboard renders these actions in a "More" (three-dots) menu within the widget toolbar, keeping the widget host-agnostic while giving authors control over exposed links.

## Impact

- **Widget authors:** Can now declare `actions` in `widget.json` to expose links or downloads without writing host-specific UI code.
- **Hosting & platform teams:** The `/wp/v2/widget-modules` REST endpoint now returns an `actions` array in each widget record; no migration or configuration changes are required.
- **Site owners:** No direct impact; existing widgets continue to function unchanged.
- **No breaking changes or deprecations.**

## Technical details

The change propagates `actions` across the full widget pipeline. On the server, `WP_Widget_Type` (in `lib/experimental/dashboard-widgets/class-wp-widget-type.php`) gains a `$actions` property, populated via `gutenberg_sanitize_widget_actions()` which validates `id`, `label`, and `href` while casting optional `download` and `openInNewTab` fields. `gutenberg_translate_widget_metadata()` runs `actions` through the i18n schema. The REST controller (`WP_REST_Widget_Modules_Controller`) exposes `actions` in `get_item_schema()` and `prepare_item_for_response()`. On the client, `useWidgetTypes()` merges the field into `WidgetType`, and the new `WidgetActions` component in `packages/widget-dashboard/src/components/widget-actions/widget-actions.tsx` renders a `Menu` of `Link` items in the toolbar controls. The build manifest pipeline (`build.mjs` `toPhpActionsLiteral`) serializes the field to PHP.

## Contribution

Opened and merged by @retrofox with co-authorship from @chihsuan. The PR landed with a minimal +11 B size impact. Review focused on UI layout (reserving space for the new trigger in inline-controls) and outlined follow-ups for `scope`, `callback`, and `steps` fulfillments.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
