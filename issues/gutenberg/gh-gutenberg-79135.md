# #79135: Fix responsive element styles front end output

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @tellthemachines
- **Labels:** `[Type] Bug`, `[Feature] Style States`
- **Merged:** [`712f3fd`](https://github.com/WordPress/gutenberg/commit/712f3fdde58828c18d769c90354f4b79419de473)
- **Discussion:** [#79135](https://github.com/WordPress/gutenberg/pull/79135) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This pull request fixes a bug where viewport-specific state styles for block elements (such as `link` or `heading`) were not being rendered on the front end. The change adds logic to `gutenberg_render_block_states_support()` to process `$style[ $breakpoint ]['elements']` and correctly output media-query scoped CSS for both root element styles and their pseudo-states. Developers using the Style States feature will now see responsive element styling applied correctly across breakpoints.

## Impact

- **Block & theme developers:** No code changes required. If you were relying on viewport-specific element styles (e.g., mobile link colors) via the block editor, they will now render correctly on the front end.
- **Site owners:** Responsive element styling applied in the editor will now match the front-end output.
- **No action required** for existing sites or plugins; this is a transparent front-end rendering fix.

## Technical details

The diff modifies `lib/block-supports/states.php` to introduce two helper functions: `gutenberg_get_block_state_element_selectors()` (which resolves block root selectors to element selectors using `WP_Theme_JSON_Gutenberg::ELEMENTS`) and `gutenberg_add_block_state_style_rule()` (which compiles state styles and attaches optional `rules_group` metadata like media queries). Inside `gutenberg_render_block_states_support()`, a new conditional block checks for `$style[ $breakpoint ]['elements']`, iterates through valid element names, resolves their selectors, and applies `gutenberg_get_root_state_style()` alongside pseudo-state iteration using `WP_Theme_JSON_Gutenberg::VALID_ELEMENT_PSEUDO_SELECTORS`. The refactored `gutenberg_get_block_state_style_rules()` now delegates to the new helper, removing inline compilation logic. A PHPUnit test in `phpunit/block-supports/states-test.php` verifies that a mobile breakpoint `link` color generates the expected `@media (width <= 480px)` scoped CSS.

## Contribution

Opened by @tellthemachines as part of the broader Style States initiative (#77817), the PR was co-authored by @ramonjd during review. While the trunk merge proceeded smoothly, the automated backport to `release/23.4` failed due to a conflict in `lib/block-supports/states.php`. During the manual backport discussion, @ramonjd noted the fix depends on `gutenberg_get_state_style_with_fallback_dimension_styles`, which hadn't shipped in the release branch yet. The team weighed waiting for the next release, backporting the dependent feature (#78795), or isolating the single missing function, ultimately leaving the backport status open pending further coordination.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
