# #78296: Guidelines: Refine access policy

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @gziolo
- **Labels:** `[Type] Enhancement`, `[Feature] Guidelines`
- **Merged:** [`05d6b08`](https://github.com/WordPress/gutenberg/commit/05d6b0800bc68912798b9a5a2fdd31370733c61b)
- **Discussion:** [#78296](https://github.com/WordPress/gutenberg/pull/78296) · 2 comments · 0 reactions

## Summary

The `wp_guideline` custom post type's access model was rewritten from static capability aliases to a runtime-synthesized policy via a `user_has_cap` filter. This ensures non-admin users can only create and manage `private` guideline posts owned by them, while administrators retain full CRUD and publishing privileges across all rows. A new `memory` guideline kind was added to the registry, and the native WordPress admin post screen was fully disabled.

## Impact

['**Plugin & AI Agent Developers**: Interacting with `wp_guideline` via REST or WP_Query now requires explicit authentication. Non-admin roles receive ambient `read_guidelines` and `edit_guidelines` capabilities but are restricted to their own `private` rows for post-specific actions. Attempting to set a status other than `private` via the REST API returns a `rest_cannot_publish` error.', '**Administrators**: Retain full access; no code changes required.']

## Technical details

- In `lib/experimental/guidelines/class-gutenberg-guidelines-post-type.php`, `register()` now sets `'show_ui' => false` and simplifies the `capabilities` map to just `'read' => 'read_guidelines'`. A new `_wp_guidelines_synthesize_caps()` callback is hooked to `user_has_cap` to dynamically assign guideline primitives based on role (`manage_options`) and post ownership/status.
- The `user_has_cap` filter grants ambient `read_guidelines` and `edit_guidelines` for Contributors, Authors, and Editors. Per-post primitives (`edit_post`, `delete_post`, `read_post`) are conditionally resolved to author-only rights when the target row is `private`.
- In `class-gutenberg-guidelines-rest-controller.php`, collection reads now gate on authentication via `get_items_permissions_check()` and apply `'perm' => 'readable'` in `prepare_items_query()` to scope pagination totals correctly. `handle_status_param()` enforces `private` for non-publishing roles, and `prepare_item_for_database()` defaults new creations to `private` instead of `draft`.
- Taxonomy capabilities for `wp_guideline_type` were tightened: `manage_terms` and `delete_terms` now require `manage_options`, while `edit_terms` and `assign_terms` require `edit_guidelines`.
- `index.php` removed the `current_screen` hook that previously disabled the block editor and media buttons for the CPT. `guidelines.php` registers `'memory'` alongside `content` in `wp_guideline_types()`.

## Contribution

Open and merged by @gziolo with co-authorship from @artpi as part of the broader content guidelines architecture (#77230). The commit replaces static CPT capability mappings with a dynamic runtime filter, hardens REST authorization checks to prevent cross-user access to scratchpad rows, and drops legacy UI scaffolding. Test suites were updated to use factory-based fixtures and added coverage for the new ownership-scoped access policy.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
