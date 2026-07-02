# #79625: View config: Add better post type default `form`

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ntsekouras
- **Labels:** `[Type] Enhancement`, `[Feature] Extensibility`, `[Package] Editor`, `[Feature] DataViews`
- **Merged:** [`bda5de3`](https://github.com/WordPress/gutenberg/commit/bda5de38e6fc7b8acd137c11daeee381988ea69d)
- **Discussion:** [#79625](https://github.com/WordPress/gutenberg/pull/79625) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This PR centralizes the default `form` view configuration for post types by introducing `_gutenberg_get_default_post_type_form()` and removing hardcoded per-type callbacks for `post` and `page`. It also refactors the editor sidebar renderer to drive DataForm Inspector toggling off a global experiment flag rather than manual post-type arrays, streamlining how the Document Summary handles conditional rendering.

## Impact

- **Plugin/Theme Developers**: Those filtering `gutenberg_get_entity_view_config()` will now receive a unified default form schema for standard post types. Custom per-type callbacks are no longer necessary unless overriding the base layout or fields.
- **Editor/Internal Developers**: The `Sidebar` component no longer selects or passes `postType`; conditional rendering of `<DataFormPostSummary />` is now driven solely by `window.__experimentalDataFormInspector`.
- No immediate migration required. Most developers can treat this as an internal consolidation, but those extending the Document Sidebar should note the shift from per-type gating to experiment-driven toggling.

## Technical details

In `lib/compat/wordpress-7.1/view-config-api.php`, `_gutenberg_get_entity_view_config_post_type_page` and `_gutenberg_get_entity_view_config_post_type` are removed. A new `_gutenberg_get_default_post_type_form()` returns a structured array containing layout definitions, fields (`featured_media`, `excerpt`, `status`, `discussion`), and child configurations. `gutenberg_get_entity_view_config()` now applies this default when `$kind === 'postType'`. The `init` hook in `gutenberg_register_entity_view_config_filters()` now conditionally registers core filters via `function_exists( $gb_callback )`, allowing plugins to safely override defaults without conflicts.

In `packages/editor/src/components/sidebar/index.js`, `Sidebar` no longer calls `select(editorStore).getCurrentPostType()`. The `postType` prop is dropped from `SidebarContent`. Inside `SidebarContent`, `const isDataFormInspectorEnabled = window?.__experimentalDataFormInspector;` replaces the previous inline array check, conditionally rendering `<DataFormPostSummary />` or `<PostSummary />`.

`packages/editor/src/components/sidebar/post-summary.js` strips its inline post-type experiment check and unconditionally renders the extracted `ClassicPostSummary`; the DataForm swap is handled upstream.

## Contribution

Opened and merged by @ntsekouras (co-authored with @mcsf and @jorgefilipecosta) as part of Gutenberg issue #76076. The discussion focused on removing per-type gating from the form config layer and centralizing logic to support the broader DataViews Inspector rollout. Merged in commit `bda5de3`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
