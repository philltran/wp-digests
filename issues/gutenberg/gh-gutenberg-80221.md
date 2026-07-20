# #80221: Notes: Arm the mention kses allowance on REST note creation

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @adamsilverstein
- **Labels:** `[Type] Bug`, `Backport to Gutenberg RC`, `Backported to WP Core`, `[Feature] Notes`
- **Merged:** [`1e2ccca`](https://github.com/WordPress/gutenberg/commit/1e2ccca230b1066fbbab364a7f6d6ad504d03c73)
- **Discussion:** [#80221](https://github.com/WordPress/gutenberg/pull/80221) · 6 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes a bug where mention markup (`<a class="wp-note-mention user-N">`) was silently stripped by kses when creating notes via the REST API as a user without `unfiltered_html`. The fix arms the mention kses allowance during REST note creation by resolving the comment type from the request payload instead of relying on an existing comment ID.

## Impact

- **Plugin & theme developers / REST API consumers:** No code changes required. Notes created via `POST /wp/v2/comments` with `type=note` will now preserve mention chips for non-admin users.
- **Editors & site owners:** Mention chips in the block editor will no longer render as plain links after saving notes.
- **No action required** for existing integrations; this is a transparent bug fix that restores expected sanitization behavior.

## Technical details

Modifies `gutenberg_notes_scope_mention_kses_rest()` in `lib/compat/wordpress-7.1/block-comments.php`. Previously, the function only resolved the comment type via `get_comment_type( (int) $request['id'] )`, which fails on creation since the ID is not yet set. The diff adds a fallback that checks the request payload:

```php
if ( '' === $comment_type && isset( $request['type'] ) ) {
    $comment_type = $request['type'];
}
```

This ensures `gutenberg_notes_arm_mention_kses()` runs before `wp_filter_comment()` sanitizes the content. The controller inserts notes directly through `wp_filter_comment()` rather than `wp_new_comment()`, bypassing the `preprocess_comment` filter path, so this REST-specific hook is the only opportunity to arm the allowance. Regression tests added in `phpunit/tests/notes-mention-kses-test.php` verify the behavior for both creation and updates.

## Contribution

Opened by @adamsilverstein and co-authored with @mamaduka. The PR was merged to trunk and subsequently backported to the `wp/7.1` branch by @t-hamano after resolving a cherry-pick conflict. Review discussion focused on labeling and backport strategy, confirming no JavaScript changes required backporting.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
