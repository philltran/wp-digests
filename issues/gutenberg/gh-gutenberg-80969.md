# #80969: Widgets: carry a declarative `icon` through the widget pipeline

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @retrofox
- **Labels:** `[Type] Enhancement`, `[Package] wp-build`, `[Feature] Dashboard`, `[Package] Widget primitives`
- **Merged:** [`0e03429`](https://github.com/WordPress/gutenberg/commit/0e034290c7817205ef42d2b60b614254d01dd917)
- **Discussion:** [#80969](https://github.com/WordPress/gutenberg/pull/80969) · 5 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Dashboard widgets can now declare their visual identifier as a serializable string (`collection/icon-name`) in `widget.json` instead of a React element in `widget.ts`. This declarative `icon` field travels through the build manifest, PHP registry, and REST API, resolving client-side via the Icons API. It eliminates the need to execute widget modules just to render header icons and enables headless consumers to read widget visuals directly from `/wp/v2/widget-modules`.

## Impact

- **Widget authors:** Can now specify `icon` as a `collection/icon-name` string in `widget.json`. The old `widget.ts` default export for icons is removed from in-tree widgets.
- **REST & headless consumers:** The `/wp/v2/widget-modules` endpoint now returns a readonly `icon` field per record.
- **Hosting & platform:** Requires the `gutenberg-dashboard-widgets` experiment to be enabled for the PHP registry and resolver to load. No migration needed for existing widgets; malformed references are silently dropped by `gutenberg_sanitize_widget_icon()`.

## Technical details

The diff introduces `icon` to `widget.json` metadata and hydrates it into `WP_Widget_Type::$icon` via `gutenberg_sanitize_widget_icon()`, which enforces a `collection/icon-name` regex shape (`#^[a-z0-9](?:[a-z0-9_-]*[a-z0-9])?/[a-z0-9](?:[a-z0-9_-]*[a-z0-9])?$#`). `WP_REST_Widget_Modules_Controller` exposes the field in its schema and `prepare_item_for_response()`. Server-side icons not in the `core` collection are registered under a new `dashboard-widgets` collection in `widget-icons.php` (e.g., `dashboard-widgets/drafts`). On the client, `@wordpress/widget-primitives` adds `registerIconResolver()` and updates `useWidgetTypes()` to resolve references through the resolver registry. `@wordpress/dashboard-init` calls `registerDashboardIconResolver()`, which fetches the SVG via `coreStore.getEntityRecord('root', 'icon', reference)`, parses it with `html-react-parser`, and applies `fill="currentColor"` for inheritance.

## Contribution

Merged by @retrofox with co-author @chihsuan, this PR implements steps 1 and 2 of a broader refactor (#80938) to declarative widget metadata. The approach replaces module-execution icon rendering with a resolver seam, allowing icons to flow through the same pipeline as other metadata fields. Follow-ups noted in the PR include supporting inline SVG markup, updating the `widget.json` schema file, and potentially promoting the temporary `dashboard-widgets` icon collection to the public `core` set.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
