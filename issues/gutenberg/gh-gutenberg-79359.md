# #79359: Post Editor: Use the correct directory for recent preload improvements

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mamaduka
- **Labels:** `[Type] Performance`, `[Package] Edit Post`
- **Merged:** [`0d2498f`](https://github.com/WordPress/gutenberg/commit/0d2498fce66f6a38c66ed4a9bbd87b5296575877)
- **Discussion:** [#79359](https://github.com/WordPress/gutenberg/pull/79359) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This pull request consolidates recent post-editor preload optimizations into the WordPress 7.1 branch, replacing a hardcoded `/wp/v2/posts` OPTIONS route with `rest_get_route_for_post_type_items()` and adding a conditional check to only preload the author when the post type supports `author`. The change improves editor startup performance by ensuring preloaded REST paths match the actual post type being edited and preventing unnecessary author requests for custom post types.

## Impact

- **Plugin & theme developers:** Developers hooking into `block_editor_rest_api_preload_paths` will now receive the `$context` parameter in the consolidated 7.1 callback, enabling more precise path generation.
- **Site owners & editors:** Faster post-editor load times due to accurate preloading and elimination of unused author requests on custom post types.
- **No action required** for existing code; the change is internal to the block editor's REST API preloading logic.

## Technical details

The diff moves preload logic from `lib/compat/wordpress-7.0/preload.php` to `lib/compat/wordpress-7.1/preload.php`, renaming the callback to `gutenberg_block_editor_preload_paths_7_1` and updating the filter registration to pass two arguments. The hardcoded `array( '/wp/v2/posts', 'OPTIONS' )` is replaced with `array( rest_get_route_for_post_type_items( $context->post->post_type ), 'OPTIONS' )`. Author preloading is now gated behind `post_type_supports( $context->post->post_type, 'author' ) && $author_id > 0`.

Filter registration changed from:
```php
add_filter( 'block_editor_rest_api_preload_paths', 'gutenberg_block_editor_preload_paths_root_fields' );
```
to:
```php
add_filter( 'block_editor_rest_api_preload_paths', 'gutenberg_block_editor_preload_paths_7_1', 10, 2 );
```

## Contribution

Opened and merged by @Mamaduka as a follow-up to #78565, this PR consolidated earlier preload improvements into the WP 7.1 branch. The implementation was adjusted to dynamically resolve the post type OPTIONS route and conditionally preload authors, ensuring the logic aligns with the final merged state of the feature set.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
