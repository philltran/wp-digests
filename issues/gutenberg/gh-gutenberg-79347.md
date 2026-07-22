# #79347: View Config API and REST Endpoint: make them core ready

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @oandregal
- **Labels:** `[Type] Enhancement`, `[Feature] DataViews`
- **Merged:** [`e2ca496`](https://github.com/WordPress/gutenberg/commit/e2ca4961c5874d8b24b3e5e0c81adddab7947907)
- **Discussion:** [#79347](https://github.com/WordPress/gutenberg/pull/79347) · 2 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

The Gutenberg plugin hardens its View Config API and `wp/v2/view-config` REST endpoint to ensure seamless compatibility when these features land in WordPress core. It fixes three JSON schema mismatches where empty arrays were serialized as `[]` instead of `{}`, adds missing layout schema properties, and consolidates scattered post-type filter overrides into a single `init`-hooked routine. This guarantees the plugin's configuration takes precedence over core's defaults without breaking or causing race conditions during REST requests.

## Impact

- **Plugin & theme developers:** No immediate code changes required. The changes are internal to the Gutenberg plugin's compatibility layer.
- **Hosting & platform teams:** No action required. The plugin registers the route with `$override = true`, ensuring it automatically replaces the core endpoint once merged.
- **Headless & REST consumers:** The `wp/v2/view-config` endpoint now strictly adheres to its documented schema. Empty `form` and `layout` fields will consistently return JSON objects (`{}`) rather than arrays (`[]`), preventing type-coercion issues in strict clients or SDKs.

## Technical details

The diff modifies `lib/compat/wordpress-7.1/class-gutenberg-rest-view-config-controller-7-1.php` and `lib/compat/wordpress-7.1/view-config-api.php`.

- **Schema/Response Fixes:** In `Gutenberg_REST_View_Config_Controller_7_1::get_items()`, the code now explicitly casts `default_view['layout']`, each entry in `default_layouts`, and `form` to `(object)` before returning the REST response. This aligns PHP's empty array serialization with the JSON schema's `type: 'object'` definition.
- **Schema Expansion:** `get_item_schema()` now includes the `layout` property in `default_view` and merges `get_list_layout_schema()['properties']` into the combined layout schema via `get_combined_layout_schema()`.
- **Route Override:** `register_routes()` passes `true` as the fourth argument to `register_rest_route()`, forcing the plugin's controller to override the core route when it exists.
- **Filter Consolidation:** The diff removes four separate `remove_filter`/`add_filter` blocks for `postType_page`, `postType_post`, `postType_wp_block`, `postType_wp_template_part`, and `postType_wp_template`. Instead, it introduces `gutenberg_register_entity_view_config_filters()`, which loops through these post types on the `init` hook, removes core's `_wp_get_entity_view_config_post_type_*` callbacks, and registers the plugin's `_gutenberg_*` replacements. This ensures core defaults are present before removal, avoiding race conditions during REST requests.
- **Tests:** Adds `phpunit/view-config-api-test.php` and `phpunit/class-gutenberg-rest-view-config-controller-test.php` covering config shape, filter overrides, permission checks, and JSON serialization.

## Contribution

Opened and merged by @oandregal as part of the DataViews initiative (#76544). The PR was authored with AI assistance for test generation and schema mismatch detection, but the author verified all changes against the PHPUnit suite. The design focused on consolidating scattered filter overrides into a single `init`-hooked routine to prevent timing issues when core registers its own callbacks. No major design debates or rejected alternatives were recorded in the PR discussion.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
