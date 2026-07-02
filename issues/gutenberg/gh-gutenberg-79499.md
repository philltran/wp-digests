# #79499: Performance: Follow-up fixes to prepend_to_selector() optimization.

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @dmsnell
- **Labels:** `[Type] Performance`
- **Merged:** [`64f2669`](https://github.com/WordPress/gutenberg/commit/64f266949fcc3b1b6e173a42734ef3094d3802c2)
- **Discussion:** [#79499](https://github.com/WordPress/gutenberg/pull/79499) · 4 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

This pull request corrects a correctness bug introduced by a recent performance optimization in `WP_Theme_JSON_Gutenberg::prepend_to_selector()`. The original fast-path incorrectly assumed that the absence of a parenthesis `(` meant a selector string contained no hidden commas, but valid CSS syntax allows commas inside strings, comments, escaped characters, and comment delimiters (`<!--`, `-->`). The fix replaces the naive check with a comprehensive delimiter scan and refines `split_selector_list()` to properly skip non-comma tokens before splitting selector lists.

## Impact

['- **Theme & plugin developers:** No code changes required. `theme.json` processing now correctly handles selectors containing commas within quotes, comments, or escape sequences without corrupting output CSS.', '- **Performance & platform teams:** The optimized `prepend_to_selector()` fast-path is now safe for all valid CSS syntax forms, preventing selector parsing failures that could break theme stylesheets or block rendering.', '- **No action required.** Developers relying on the public block editor experience will automatically benefit from corrected `theme.json` parsing performance.']

## Technical details

In `lib/class-wp-theme-json-gutenberg.php`, the fast-path gate in `prepend_to_selector()` now uses `strcspn( $selector, '/\"(<\\' )` to detect `/`, `'`, `\"`, `<`, `(`, and `\`. This ensures the string-replacement shortcut only triggers when it is mathematically safe to process commas as top-level delimiters.
The previously paired method `split_selector_list()` was rewritten from a simple `for` loop tracking `$parentheses_depth` to a jump-based `while` loop using `strcspn()`. It now explicitly handles:
- Escaped characters (skipping the next byte when `\` is found)
- Parenthesized expressions (tracking depth and jumping past closing `)`)
- Strings (`'` or `"`, handling internal escapes, advancing to the closing quote)
- Comments (`/* ... */`) and Comment Delimiters (`<!--`, `-->` / CDO/CDC)
A new PHPUnit test file `phpunit/class-wp-theme-json-selector-list.php` was added with data providers validating these edge cases (e.g., `[data-label="Save, continue"]`, `:where(.a, .b)`, escaped commas).

## Contribution

Opened and merged by `@dmsnell` with co-authorship from `@westonruter`. The PR serves as a direct follow-up to #76556, addressing the observation that the initial optimization missed several CSS syntax forms that can hide commas. After brief validation where the author ran automated generation of ~15M selectors to verify correctness, the commits were squashed and merged into `wp-env`.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
