# #80016: RTC: Disable Quick Edit while a post is being edited

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @shekharnwagh
- **Labels:** `[Type] Enhancement`, `[Feature] Real-time Collaboration`, `No Core Sync Required`
- **Merged:** [`302b7e0`](https://github.com/WordPress/gutenberg/commit/302b7e0a637e3a1d513fb23e28fb8f5bd812cc85)
- **Discussion:** [#80016](https://github.com/WordPress/gutenberg/pull/80016) · 3 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Real-time collaboration (RTC) in Gutenberg now disables the Quick Edit action on the Posts list while a post has an active editing session. This prevents non-collaboration-aware inline edits from silently diverging from or overwriting content in an active editor tab. The behavior applies only when RTC is enabled for the site and post type.

## Impact

- **Plugin & theme developers / Admin UI customizers:** No public API changes, but custom admin screens that rely on Quick Edit alongside RTC will see the action hidden on rows marked as locked. Bulk-edit checkboxes remain enabled.
- **Site owners & editors:** Quick Edit is automatically disabled on rows showing "Currently being edited" when RTC is active, preventing accidental overwrites.
- **No action required** for existing code; the blocking logic is handled server-side and via CSS overrides.

## Technical details

- Adds `gutenberg_get_active_edit_lock_user( $post_id )` in `lib/compat/wordpress-7.1/collaboration.php` to parse fresh `_edit_lock` metadata, including locks owned by the current user, while ignoring deleted users and respecting the `wp_check_post_lock_window` filter (default 150s).
- Hooks `gutenberg_block_quick_edit_for_active_lock()` to `wp_ajax_inline-save` at priority `0`. If the current user holds a fresh lock, the function calls `wp_die()` with a clear error message. If no lock exists, it attaches a filter to `update_post_metadata` that returns `false` for `_edit_lock` writes, preventing Core from creating a false-positive lock that would block subsequent Quick Edits.
- Updates `gutenberg_filter_locked_posts_heartbeat_for_rtc()` to accept a `$data` parameter and inject generic "Currently being edited" states for the current user's own fresh locks.
- Removes the CSS rule `tr.wp-locked .row-actions .inline { display: revert; }` from `gutenberg_post_list_collaboration_styles()`, ensuring Quick Edit stays hidden on locked rows while bulk-edit checkboxes remain visible.
- Includes PHPUnit tests in `phpunit/tests/collaboration/quickEditCollaborationLock.php` and `postListCollaborationStyles.php`.

## Contribution

Opened by @shekharnwagh and merged after review by @alecgeatches, @pkevan, and others. The initial implementation removed the `_edit_lock` post meta after a Quick Edit save, but reviewers flagged potential race conditions and noted that Core does not normally delete edit locks. The author revised the approach to prevent the lock write entirely via the `update_post_metadata` filter, which was accepted and merged.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
