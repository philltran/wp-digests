# #79452: Experimental: Expand DataForm inspector to patterns

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @jorgefilipecosta
- **Labels:** `[Package] Editor`, `[Type] Feature`, `[Package] Edit Site`, `[Package] Fields`
- **Merged:** [`1e094de`](https://github.com/WordPress/gutenberg/commit/1e094dedb86b0adb1c99d682678e0b7b64b2dacf)
- **Discussion:** [#79452](https://github.com/WordPress/gutenberg/pull/79452) · 6 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Expands the experimental `Editor Inspector: Use DataForm` feature to WordPress block patterns (`wp_block`). This change introduces dedicated client-side fields for pattern descriptions and sync status, registers a new server-side view configuration, and aligns the `wp_block` post summary with the unified DataForm architecture used across the site editor.

## Impact

- **Internal package & component maintainers**: The `patternStatusField` has been moved out of `edit-site/src/components/page-patterns/fields.js` into `@wordpress/fields`. External packages importing it directly will need to update their import paths to `@wordpress/fields`.
- **Pattern developers using the Site Editor**: No immediate code changes required. UI behavior shifts toward a structured DataForm layout when the experiment is enabled, improving consistency with posts and templates.
- **No action required** for standard site editors or pattern consumers relying on default block editor behavior.

## Technical details

- Registers a server-side view configuration in `lib/compat/wordpress-7.1/view-config-api.php` via `_gutenberg_get_entity_view_config_post_type_wp_block()`, defining a `form` layout with `excerpt` and `sync-status` fields alongside shared revision/info fields.
- Centralizes pattern field logic into two new exports in `packages/fields/src/fields/index.ts`: `patternDescriptionField` (maps to `item.excerpt` for reading/writing via a textarea control) and `patternSyncStatusField` (renders read-only status based on `item.wp_pattern_sync_status` or fallback meta, supporting 'Synced'/'Not synced' filters).
- Updates `packages/editor/src/dataviews/store/private-actions.ts` to conditionally attach these fields for `postType === 'wp_block'` in `registerPostTypeSchema()`.
- Adds `'wp_block'` to the post type array in `packages/editor/src/components/sidebar/post-summary.js` triggering `DataFormPostSummary`.
- Refactors `packages/edit-site/src/components/page-patterns/index.js` to replace manual field concatenation with `usePostFields({ postType })`, eliminating duplication and simplifying the render pipeline.

## Contribution

Opened and merged by @jorgefilipecosta, with review from @ntsekouras. Part of a broader effort to unify the DataForm inspector across entity types (issue #76076). During review, the team decided to move `patternStatusField` out of the `edit-site` package into the shared `@wordpress/fields` module to prevent duplication with template/part editors and establish a reusable base for all pattern-specific fields.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
