# The notify_post_author filter now has the final say on post author notifications

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Adam Silverstein
- **Published:** 2026-08-05
- **Tags:** `General`, `7.1`, `dev-notes`, `dev-notes-7-1`
- **Link:** [https://make.wordpress.org/core/2026/08/05/the-notify_post_author-filter-now-has-the-final-say-on-post-author-notifications/](https://make.wordpress.org/core/2026/08/05/the-notify_post_author-filter-now-has-the-final-say-on-post-author-notifications/)
- **Usefulness:** 4/5

## Summary

WordPress 7.1 changes the execution order in `wp_new_comment_notify_postauthor()` so that the `notify_post_author` filter runs after the comment’s approval status is evaluated. Previously, the filter received a default based solely on the `comments_notify` option and its return value was silently ignored for unapproved comments. Now, the filter receives an accurate default reflecting approval status, and its return value definitively controls whether the author notification is sent.

## Impact

- **Plugin & theme developers:** Callbacks using `__return_true` on `notify_post_author` to force notifications will now also trigger emails for comments held in moderation, marked as spam, or trashed. To maintain previous behavior, filter callbacks must explicitly check the comment’s approval status.
- **Suppressing notifications:** Callbacks that return `false` to block notifications are unaffected.
- **Site owners & platform teams:** No direct action required unless a custom plugin relies on the old filter behavior.

## Technical details

The change modifies `wp_new_comment_notify_postauthor()` to evaluate `comment_approved` before applying the `notify_post_author` filter. The default value passed to the filter is now strictly `true` or `false` (previously could be a raw string like `'1'` from the `comments_notify` or `wp_notes_notify` options). The filter’s return value is now final; returning `true` will send a notification even for unapproved comments. Additionally, if the provided comment ID does not resolve to a valid comment object, the function returns `false` immediately without firing the filter.

```php
// Before (filter ignored for unapproved comments)
add_filter( 'notify_post_author', '__return_true' );

// After (explicitly check approval status to avoid spam/trashed notifications)
add_filter( 'notify_post_author', function ( $maybe_notify, $comment_id ) {
    $comment = get_comment( $comment_id );
    if ( $comment && '1' === $comment->comment_approved ) {
        return true;
    }
    return $maybe_notify;
}, 10, 2 );
```

## Contribution

Authored by Adam Silverstein and peer-reviewed by @milana_cap, the change addresses ticket #64217 by reordering the approval check to align the filter’s default value with actual notification routing. The discussion focused on correcting the discarded return value for unapproved comments, with no alternative approaches retained.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
