# #79911: RTC: Fix empty revision from peer autosave

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @alecgeatches
- **Labels:** `[Type] Bug`, `[Feature] Real-time Collaboration`, `No Core Sync Required`
- **Merged:** [`f5dae78`](https://github.com/WordPress/gutenberg/commit/f5dae78d2057a074976afd9f5c325dfbf8278013)
- **Discussion:** [#79911](https://github.com/WordPress/gutenberg/pull/79911) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Real-time collaboration (RTC) autosaves were creating redundant, "blank" revisions when multiple users edited the same draft. This PR fixes the autosave controller to compare incoming autosaves against the most recent revision instead of the stale parent post, preventing duplicate no-op revisions from being stored.

## Impact

- **RTC users & editors:** Eliminates "blank" autosave revisions in the revisions UI during collaborative editing sessions.
- **Plugin & theme developers:** No public API changes, deprecations, or breaking changes. The modification is internal to the Gutenberg REST autosave controller.
- **Action required:** None. The fix applies automatically when Real-Time Collaboration is enabled and is fully backward compatible.

## Technical details

The diff modifies `lib/compat/wordpress-7.1/class-gutenberg-rest-autosaves-controller.php`. The private method `is_redundant_autosave()` previously accepted a `$user_id` parameter and early-returned `false` if `wp_get_post_autosave( $post->ID, $user_id )` existed, comparing incoming data against the parent `$post`. The fix removes the `$user_id` parameter and the per-user check. It now calls a new private method, `get_autosave_comparison_baseline( $post )`, which returns the latest revision when `wp_is_collaboration_enabled()` is true and the revision's `post_modified_gmt` is greater than or equal to the parent's. The comparison logic (`_wp_post_revision_fields()` and `get_metadata_raw()`) now operates against this `$baseline` instead of the parent post, ensuring peer autosaves matching the converged CRDT state are correctly identified as redundant.

## Contribution

Opened and merged by @alecgeatches, with review contributions from @ingeniumed. The PR addresses a follow-up to #79591, which fixed single-user empty autosaves but didn't account for multi-user RTC workflows. The author initially questioned the UX of the autosave notice but separated the server-side fix from the frontend notice logic, merging only the backend deduplication change. Tests were added to `phpunit/tests/collaboration/restAutosavesController.php` and a new Playwright spec `collaboration-peer-blank-revision.spec.ts`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
