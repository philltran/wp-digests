# #78217: Block supports: Optimize custom CSS class rendering and parsing

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @westonruter
- **Labels:** `Customization`, `[Type] Performance`, `Backported to WP Core`
- **Merged:** [`ab84867`](https://github.com/WordPress/gutenberg/commit/ab84867a5fd6c2a657f5f34444d7db44b3d38591)
- **Discussion:** [#78217](https://github.com/WordPress/gutenberg/pull/78217) · 5 comments · 1 reactions

## Summary

Optimizes internal block support logic for rendering custom CSS classes, reducing rendering overhead and improving parsing accuracy. The change moves early string validation before the `WP_Block_Type_Registry` lookup to skip unnecessary registry lookups when no CSS is present, and replaces regex-based class extraction with a fast-path `str_contains()` check plus explicit tokenization via `strtok()`. This keeps the block rendering pipeline faster while eliminating false-positive matches on unrelated classes.

## Impact

- **Theme & plugin developers**: No breaking changes or migration steps required. Internal functions in the block supports system were optimized, but their public signatures and fallback behaviors remain unchanged.
- **Block authors using `customCSS` support**: Experience slightly faster rendering due to early exits and avoided regex compilation. Class extraction is now more robust against malformed or prefixed class strings.
- **Hosting & platform teams**: Transparent runtime optimization. No server configuration or cache invalidation changes needed.
- **Action required**: None. Safe backport with zero developer impact.

## Technical details

- **File**: `lib/block-supports/custom-css.php`
- **Early Exit Optimization**: `gutenberg_render_custom_css_support_styles()` previously called `WP_Block_Type_Registry::get_instance()->get_registered()` immediately on entry. The diff reorders logic to perform an early `is_string()` and trim guard on `$parsed_block['attrs']['style']['css'] ?? null` first, preventing expensive registry lookups for blocks that lack CSS attributes.
- **Parsing Swap**: Replaced `preg_match( '/\bwp-custom-css-\S+\b/', $class_string, $matches )` with a `str_contains()` fast path followed by manual tokenization via `strtok()`. This removes per-call regex compilation overhead and prevents false positives where an unrelated class merely contains the `wp-custom-css-` substring.
- **Type Safety**: Added explicit `@phpstan-param` array shapes for `$parsed_block` and `$block` parameters to improve static analysis coverage.

```php
// Before (regex-based extraction)
$class_string = $block['attrs']['className'] ?? '';
preg_match( '/\bwp-custom-css-\S+\b/', $class_string, $matches );

// After (fast-path + token iteration)
$class_name_attr = $block['attrs']['className'] ?? null;
if ( ! is_string( $class_name_attr ) || ! str_contains( $class_name_attr, 'wp-custom-css-' ) ) {
    return $block_content;
}
$custom_class_name = strtok( $class_name_attr, " \t\f\r\n" );
while ( false !== $class_token ) { ... }
```

## Contribution

Opened by @westonruter as a synchronization and follow-up to `wordpress-develop` PR #11686. Co-authored by @ramonjd and @t-hamano, the changes were backported to `wp/7.0` for distribution consistency. Discussion confirmed the optimization is purely internal and does not alter external Core behavior, focusing instead on standardizing block support parsing logic across the monorepo.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
