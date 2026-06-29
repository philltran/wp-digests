# #78984: RTC: Allow disabling collaboration by post type

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @maxschmeling
- **Labels:** `[Type] Enhancement`, `[Package] Core data`, `[Package] Editor`, `[Feature] Real-time Collaboration`, `No Core Sync Required`
- **Merged:** [`e11c078`](https://github.com/WordPress/gutenberg/commit/e11c0788f0116409c32076fcc19ef7cb3dc08209)
- **Discussion:** [#78984](https://github.com/WordPress/gutenberg/pull/78984) · 5 comments · 0 reactions
- **Usefulness:** 4/5

## Summary

Real-Time Collaboration (RTC) can now be disabled for specific post types without requiring a global toggle change. Previously, enabling collaboration at the site level applied to all synced post type entities; this change introduces a filter and associated UI updates to opt out individual post types from collaborative editing and background syncing.

## Impact

- **Plugin & theme developers / site owners**: Can disable RTC for specific post types by hooking `wp_is_post_type_collaboration_disabled`. Useful when certain custom post types should operate independently of the collaborative editing engine.
- **No breaking changes**: Existing collaboration settings remain unaffected unless explicitly filtered. No migration steps required.

## Technical details

- `lib/compat/wordpress-7.1/collaboration.php` adds `wp_is_post_type_collaboration_disabled( $post_type )`, which defaults to `false` and passes `$disabled` and `$post_type` through the `wp_is_post_type_collaboration_disabled` filter.
- The same file modifies `gutenberg_inject_real_time_collaboration_setting()` to map all REST-enabled post types against the filter, injecting the resulting disabled slugs into `window._wpCollaborationDisabledPostTypes`.
- In `packages/core-data/src/entities.js`, post type entity configs receive a new `shouldSync` callback that returns `false` if the slug exists in the disabled array.
- `packages/editor/src/store/private-selectors.js` updates `isCollaborationEnabledForCurrentPost()` to evaluate `syncConfig.supportsPersistence && window._wpCollaborationEnabled && false !== syncConfig.shouldSync(...)`, ensuring the editor UI and transport layer respect the opt-out.
- `gutenberg_post_list_collaboration_row_actions()` now gates the 'Join' row action, returning unmodified default actions when collaboration is disabled for the post type.

## Contribution

Merged by @maxschmeling with co-authors @pkevan and @alecgeatches. During review, @alecgeatches flagged that the block editor list table still rendered a 'Join' row action for disabled post types, which prompted an immediate patch within the same PR to gate the collaboration row actions via `wp_is_post_type_collaboration_disabled()`. The merged diff includes dedicated PHPUnit tests for the PHP row actions and Jest test suites covering the entities loader and private selectors.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
