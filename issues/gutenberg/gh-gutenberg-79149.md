# #79149: Knowledge: Rename the Guidelines CPT storage primitive to Knowledge

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @gziolo
- **Labels:** `[Type] Breaking Change`, `No Core Sync Required`, `[Feature] Guidelines`
- **Merged:** [`6c99f5e`](https://github.com/WordPress/gutenberg/commit/6c99f5e3d0b4ca2d8fb2ff253908e343b5ebc4ce)
- **Discussion:** [#79149](https://github.com/WordPress/gutenberg/pull/79149) · 9 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This pull request renames the experimental Guidelines content storage primitive to a "Knowledge" namespace across the Gutenberg plugin. It updates the custom post type (`wp_guideline` → `wp_knowledge`), associated taxonomy (`wp_guideline_type` → `wp_knowledge_type`), REST routes, capability filters, and built-in type slugs. The consolidation prepares the feature for potential graduation without altering its underlying behavior or data shape.

## Impact

- **Plugin & Headless Developers:** Code querying `wp_guideline`, targeting `/wp/v2/guidelines`, or filtering `wp_guideline_types` will break until migrated to their `knowledge` equivalents. Capability checks must shift from `*_guidelines` to the new `_knowledge_items` namespace (plural for primitives, singular for meta).
- **Gutenberg Experiment Consumers:** No data migration is included; existing rows remain under the old CPT until manually updated. The feature remains behind the `gutenberg-guidelines` experiment flag and carries no backward-compatibility guarantee.
- **Action Required:** Update REST client paths to `wp/v2/knowledge`, replace hard-coded `wp_guideline` references, and adjust capability filters to use `wp_knowledge_types`. The Settings UI, singleton controller (`/wp/v2/content-guidelines`), and `_guideline_*` post meta keys remain unchanged and require no updates.

## Technical details

- Moves `lib/experimental/guidelines/` to `lib/experimental/knowledge/`, renaming PHP files and updating class names (e.g., `Gutenberg_Guidelines_Post_Type` → `Gutenberg_Knowledge_Post_Type`).
- Updates the post type constant `POST_TYPE` to `wp_knowledge` and taxonomy constant `TAXONOMY` to `wp_knowledge_type`. The taxonomy is explicitly marked headless (`show_ui => false`).
- Replaces the collection REST route from `/wp/v2/guidelines` to `/wp/v2/knowledge`. Collection reads now default to `status=publish`, meaning newly created items (saved as `draft` or `private`) require explicit query parameters like `?status=private` to appear.
- Renames the capability synthesis function `_wp_guidelines_synthesize_caps()` to `_wp_knowledge_synthesize_caps()`. The filter hook shifts from `wp_guideline_types` to `wp_knowledge_types`. Per-post capabilities (`edit_knowledge_item`, `delete_knowledge_item`) are now mapped to primitive caps (`edit_knowledge_items`, `delete_knowledge_items`) via `map_meta_cap()`.
- Autosaves are explicitly disabled for the new CPT while retaining revisions. Test assertions were added to verify autosave routes are unregistered and revision routes remain intact.

## Contribution

Opened and merged by `@gziolo`, with co-authors `@jorgefilipecosta` and `@artpi`. The PR was created as an illustrative implementation of a consolidation proposal to ground the ongoing discussion in working code. During review, `@gziolo` synced the branch with parallel core infrastructure work (`WordPress/wordpress-develop#12201`), incorporating refinements such as standardizing error codes from `rest_forbidden` to `rest_cannot_read`, clarifying default status behavior for REST queries, and dropping the underscore prefix from private helper functions. Scope was explicitly limited to the Gutenberg plugin experiment, with follow-up types (`skill`) and ability-management API deferred.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
