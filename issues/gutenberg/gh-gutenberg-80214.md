# #80214: Block Supports: Improve handling of block class name to avoid fatal

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @aduth
- **Labels:** `[Type] Bug`, `Global Styles`, `[Feature] Design Tools`
- **Merged:** [`d5c9b02`](https://github.com/WordPress/gutenberg/commit/d5c9b0218d71053adafc6c14f3f3c8739c151e1f)
- **Discussion:** [#80214](https://github.com/WordPress/gutenberg/pull/80214) · 9 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This pull request adds defensive type checking to block supports rendering logic to prevent PHP fatals when the `className` block attribute contains a non-string value (such as an array) due to corrupted block content. Previously, rendering functions would fail when passed malformed attributes. The fix ensures these functions gracefully return the original block content when `className` is not a string, maintaining frontend stability without altering normal block behavior.

## Impact

- **Site owners & editors**: No action required. Posts with corrupted block markup that previously caused PHP fatals will now render safely.
- **Plugin & theme developers**: No action required. The change is strictly defensive and does not modify public APIs, block registration patterns, or expected attribute shapes.
- **Hosting & platform teams**: Recommended for inclusion in point releases to prevent frontend crashes on sites with legacy or corrupted block data.

## Technical details

The diff modifies two files in the block supports registry to validate `$block['attrs']['className']` before passing it to string functions:

- `lib/block-supports/block-style-variations.php`: Extracts the attribute into `$block_class_name` and wraps the `preg_match` call with an `is_string()` guard.
- `lib/block-supports/layout.php`: Replaces the null-coalescing assignment with an explicit type check using a ternary operator.

Before/after pattern in `block-style-variations.php`:
```php
// Before
preg_match( '/\bis-style-(\S+?--\d+)\b/', $block['attrs']['className'], $matches );

// After
$block_class_name = $block['attrs']['className'];
if ( ! is_string( $block_class_name ) ) {
    return $block_content;
}
preg_match( '/\bis-style-(\S+?--\d+)\b/', $block_class_name, $matches );
```

Both `gutenberg_render_block_style_variation_class_name` and `gutenberg_render_layout_support_flag` now skip style/layout processing when `className` is an array or missing, returning the unmodified `$block_content`. Unit tests were added to `phpunit/block-supports/block-style-variations-test.php` and `phpunit/block-supports/layout-test.php` to verify behavior with non-string and missing `className` attributes.

## Contribution

Opened and authored by @aduth, with co-authors @ciampo, @tyxla, @tellthemachines, @westonruter, and @MaggieCabrera. The PR was merged into the Gutenberg repository and immediately cherry-picked to the `release/23.5` branch. It was subsequently backported to WordPress Core via PR #12516 (SVN changeset 62732) to address Trac ticket #65625, with maintainers agreeing it warranted a point release due to the stability impact on corrupted content.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
