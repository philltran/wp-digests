# REST API: Restore the global post after preparing a revision.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Weston Ruter
- **Committed:** 2026-07-31
- **Commit:** [`593166d816`](https://github.com/WordPress/wordpress-develop/commit/593166d816dbec24a7a0780fa7b675e32e504b7b)
- **Usefulness:** 4/5

## Summary

The REST API revision controller now restores the global `$post` after preparing a revision response. Previously, `WP_REST_Revisions_Controller::prepare_item_for_response()` called `setup_postdata()` without restoring the original global, causing the modified `$post` to leak into the rest of the request. This bug could cause the block editor to initialize with the wrong post when preloading autosaves. The fix captures the previous global post, restores it on all code paths, and introduces a dedicated `restore_post_data()` helper.

## Impact

- **Plugin & theme developers:** No breaking changes. If your code relies on the global `$post` remaining set to a revision after a REST request (highly unusual), it will now be restored to its prior state.
- **Block editor users & developers:** Fixes a bug where pending autosaves could cause the editor to load with incorrect post data or rewrite the URL. No action required.
- **REST API consumers:** Behavior is now consistent; the global `$post` is no longer mutated by revision/autosave endpoints. No migration needed.

## Technical details

The change modifies `src/wp-includes/rest-api/endpoints/class-wp-rest-revisions-controller.php`. In `prepare_item_for_response()`, the method now captures `$previous_post = isset( $GLOBALS['post'] ) && $GLOBALS['post'] instanceof WP_Post ? $GLOBALS['post'] : null;` before calling `setup_postdata()`. It then calls the new private method `restore_post_data( $previous_post )` on every return path, including the early return for `HEAD` requests. The `restore_post_data()` method either restores the captured post via `$GLOBALS['post'] = $previous_post; setup_postdata( $previous_post );` or, if none existed, calls `wp_reset_postdata()` followed by `$GLOBALS['post'] = null;` to prevent the main query from repopulating it. The `$post` variable inside the method is intentionally kept local to prevent filters like `the_content` from altering which revision is being prepared. Docblocks for `prepare_item_for_response()` are updated to reflect the `@since 7.1.0` restoration behavior.

## Contribution

Developed in PR #12248 and merged as a follow-up to earlier revision handling work (r40601, r59899), this fix addresses ticket #65495. During implementation, the decision was made to keep the revision object in a local variable rather than relying on the global `$post`, ensuring that filters like `the_content` cannot accidentally retarget the prepared fields. Props were given to micahele, gusgomezpg, wildworks, and dhrupo for testing and feedback.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
