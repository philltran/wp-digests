# #80771: Block supports: Return from layout support before resolving global settings

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @ramonjd
- **Labels:** `[Type] Bug`, `[Feature] Layout`, `Backported to WP Core`
- **Merged:** [`bcac35f`](https://github.com/WordPress/gutenberg/commit/bcac35fc04e65739d25e3838cdf968fd8f0a08a2)
- **Discussion:** [#80771](https://github.com/WordPress/gutenberg/pull/80771) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Fixes a fatal memory exhaustion bug in `gutenberg_render_layout_support_flag()` that occurs when plugins render blocks inside a `the_posts` callback. The issue was introduced when a global settings lookup was moved above an early-return guard, removing the base case for recursive block rendering. The fix restores the early return before the settings resolution, preventing infinite recursion and front-end crashes.

## Impact

- **Plugin & theme developers:** No breaking changes. Fixes a crash for sites that render blocks inside `the_posts` (e.g., to collect asset dependencies).
- **Hosting & platform:** Resolves `Allowed memory size ... exhausted` fatal errors on front-end requests when global styles are active and `the_posts` triggers block rendering.
- **No action required** for standard sites; the fix is automatically applied in the next WordPress/Gutenberg release.

## Technical details

- **File:** `lib/block-supports/layout.php`
- **Function:** `gutenberg_render_layout_support_flag()`
- **Change:** Moves the early return guard `if ( ! $block_supports_layout && empty( $style_attr ) ) { return $block_content; }` above the `gutenberg_get_global_settings()` call.
- **Behavior:** PR #79104 moved the guard up to access `$viewport_child_layouts`, but this broke the recursion base case for blocks with `blockName === null` (like the parsed `wp_global_styles` post content). The fix re-inserts the guard above the lookup, ensuring blocks that cannot produce layout output exit immediately without triggering the `WP_Query` for user global styles.
- **Test:** Added `test_layout_support_flag_returns_early_before_resolving_global_settings()` to `phpunit/block-supports/layout-test.php`, verifying that `wp_theme_json_data_user` is not resolved for non-layout blocks and that layout blocks still resolve settings.

## Contribution

Opened and merged by @ramonjd with co-authors @andrewserong and @fushar. The PR was quickly reviewed and backported to both the `wp/7.1` and `release/23.6` branches. During review, @andrewserong refined the manual reproduction snippet to scope the `the_posts` callback to the main query and `wp_global_styles` post type, preventing an unrelated infinite loop with Query Loop blocks that would otherwise mask the actual bug.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
