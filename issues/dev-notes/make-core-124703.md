# Media Library infinite scrolling is now enabled by default, with a per-user opt-out

- **Source:** Make WordPress Core
- **Type:** Blog post
- **Author:** Marin Atanasov
- **Published:** 2026-07-23
- **Tags:** `Core`, `7.1`, `dev-notes`, `dev-notes-7-1`, `media`, `media library`, `media-grid`
- **Link:** [https://make.wordpress.org/core/2026/07/23/media-library-infinite-scrolling-is-now-enabled-by-default-with-a-per-user-opt-out/](https://make.wordpress.org/core/2026/07/23/media-library-infinite-scrolling-is-now-enabled-by-default-with-a-per-user-opt-out/)
- **Usefulness:** 4/5

## Summary

In WordPress 7.1, infinite scrolling in the Media Library grid and Media Modal is now enabled by default, reversing the `false` default introduced in 5.8. This change removes the "Load more" button friction for users managing large libraries while preserving a per-user opt-out. The behavior is controlled by the `media_library_infinite_scrolling` filter, which now defaults to `true`, and respects a new user profile setting.

## Impact

- **Plugin & theme developers:** The `media_library_infinite_scrolling` filter now defaults to `true`. If your code relied on infinite scrolling being off by default, you must explicitly return `false` to restore the previous behavior.
- **Site administrators & editors:** Users will see infinite scrolling enabled automatically. Those who prefer the "Load more" button can disable it via a new checkbox on the Profile screen.
- **Hosting & platform teams:** No configuration changes required. The change is opt-out at the user level and filter-overridable.
- **Headless & REST consumers:** No impact; this change is strictly UI/frontend behavior in the admin media grid and modal.

## Technical details

The change modifies the default return value of the `media_library_infinite_scrolling` filter from `false` to `true` in WordPress 7.1 (#65564). The effective state is resolved in strict precedence order: a hooked filter callback overrides everything, followed by the user's stored preference, then the new `true` default. The user preference is persisted as a user meta key named `infinite_scrolling` (stored as the string `'true'` or `'false'`) and is rendered/saved via `user-edit.php` and `edit_user()`. Frontend initialization reads this value inside `wp_enqueue_media()` before applying the filter. To force a specific state regardless of user preference, developers can hook the filter:

```php
// Force infinite scrolling off for all users
add_filter( 'media_library_infinite_scrolling', '__return_false' );
```

The filter signature remains unchanged, ensuring backward compatibility with existing callbacks.

## Contribution

The provided source contains no design debate or alternative approaches; it simply credits Marin Atanasov as the author and lists six reviewers, with the precedence model (filter > user preference > default) documented as the final implementation.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
