# #78282: Block Supports: Prevent Additional CSS duplication inside Query Loop

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @Mustafabharmal
- **Labels:** `[Feature] Block API`, `[Type] Performance`, `Backport to WP Minor Release`
- **Merged:** [`f6f86c9`](https://github.com/WordPress/gutenberg/commit/f6f86c9b9ccebf3d20ef64d5bcec4a8680aa17b0)
- **Discussion:** [#78282](https://github.com/WordPress/gutenberg/pull/78282) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Prevents duplicate inline CSS output when blocks with Additional Custom CSS are rendered multiple times inside a Query Loop. The change ensures identical CSS rules are injected into the DOM exactly once, eliminating redundant markup and reducing render payload for looped templates.

## Impact

['- **Block & theme developers:** No code changes required. Sites using Query Loop or similar block patterns will automatically output cleaner, deduplicated CSS in the front end upon update.', '- **Performance/Platform teams:** Reduces DOM bloat and CSS parser overhead in loops by preventing repeated `wp_add_inline_style()` calls for identical processed content.']

## Technical details

In `lib/block-supports/custom-css.php`, the `gutenberg_render_custom_css_support_styles()` callback now deduplicates styles before injection. It registers the `wp-block-custom-css` handle only if it is not already registered, then inspects the `after` queue via `wp_styles()->get_data( $handle, 'after' )`. If `$processed_css` already exists in that array, the inline injection is skipped. The block's `attrs.className` remains updated per render to preserve correct selector targeting.

## Contribution

Opened by @Mustafabharmal and merged after review from @ramonjd, @t-hamano, @westonruter, and @benniledl. The PR was labeled for backport to the upcoming WP minor release. Reviewers validated the approach as a contained fix, noting that future style engine iterations could abstract this deduplication into a structured CSS processor.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
