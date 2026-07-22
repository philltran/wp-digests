# #76556: prepend_to_selector: optimized with str_replace()

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @josephscott
- **Labels:** `[Type] Performance`, `Global Styles`
- **Merged:** [`3f6e2c0`](https://github.com/WordPress/gutenberg/commit/3f6e2c02542ddc972f661676743767e859747f30)
- **Discussion:** [#76556](https://github.com/WordPress/gutenberg/pull/76556) · 5 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

The `prepend_to_selector()` method in `WP_Theme_JSON_Gutenberg` has been optimized to reduce CPU and memory overhead during Global Styles processing. By introducing a fast-path that uses `str_replace()` for selectors without parentheses, the method now bypasses array splitting and looping in the majority of cases. This yields a ~23% performance improvement during WordPress bootstrap and slightly reduces memory consumption.

## Impact

- **Plugin & theme developers / platform teams**: No direct code changes required. The optimization runs automatically when processing `theme.json` or Global Styles.
- **Site owners**: Indirect benefit via faster admin/dashboard load times when Global Styles are active.
- **No breaking changes or deprecations**. Existing CSS selector handling remains functionally identical.

## Technical details

The change modifies `WP_Theme_JSON_Gutenberg::prepend_to_selector()` in `lib/class-wp-theme-json-gutenberg.php`. Previously, the method always split comma-separated selectors via `static::split_selector_list()` and iterated to prepend the scope. The updated implementation first checks `str_contains( $selector, '(' )`. If parentheses are absent, it uses `str_replace( ',', ',' . $to_prepend, $selector )` to inject the prefix directly. If parentheses are present (e.g., `:where()`, `:is()`), it falls back to the original split-and-loop logic to avoid incorrectly splitting internal commas. Comprehensive PHPUnit tests were added to `phpunit/class-wp-theme-json-test.php` covering single selectors, compound selectors, whitespace preservation, and nested pseudo-class functions.

## Contribution

The author benchmarked the method across 4,000+ bootstrap calls, identified the `str_replace` optimization via AI-assisted analysis, and generated a full test suite to prevent regressions. Reviewers verified correctness and approved the merge, noting only a minor delay while CI tests stabilized. No alternative approaches were debated or rejected during the review thread.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
