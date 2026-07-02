# #79723: CSS: Follow-up fixes to split_selector_list()

- **Source:** WordPress/gutenberg
- **Type:** Pull request
- **Author:** @dmsnell
- **Labels:** `[Type] Enhancement`, `Backport from WordPress Core`
- **Merged:** [`e36bcc1`](https://github.com/WordPress/gutenberg/commit/e36bcc18352bec0cdff785e3178539130a3c9e3f)
- **Discussion:** [#79723](https://github.com/WordPress/gutenberg/pull/79723) · 2 comments · 0 reactions
- **Usefulness:** 3/5

## Summary

Follow-up changes to `split_selector_list()` in `class-wp-theme-json-gutenberg.php` that standardize whitespace handling, add a fast-path optimization for simple CSS selectors, and eliminate redundant manual trimming by callers. Ensures generated CSS selector lists consistently use `, ` spacing and improves parsing performance.

## Impact

- **Block theme developers & Theme JSON API users**: Generated CSS will now consistently include a space after commas in selector lists (e.g., `.a, .b` instead of `.a,.b`), aligning output with modern formatting standards.
- **Developers using internal Theme JSON methods**: Callers no longer need to manually `trim()` split results; the method guarantees trimmed outputs, reducing boilerplate and potential edge-case bugs in selector manipulation.
- No immediate migration or config changes required. Existing selectors remain functionally equivalent, but formatting is normalized.

## Technical details

In `lib/class-wp-theme-json-gutenberg.php`:
- `split_selector_list()` now explicitly returns `array( trim( $selector, " \t\n" ) )` for single-selector inputs and trims the final chunk in the main loop. This removes the need for callers like `scope_selector()` and block style variation handlers to manually call `trim()` on split arrays.
- `append_to_selector()` and `prepend_to_selector()` gain a fast-path: if a selector lacks `/` `'` `"` `(` `<` or `\` characters (verified via `strlen( $selector ) === strcspn( $selector, ...)`), it bypasses splitting and uses `str_replace(',', $to_append . ',', $selector)` for direct substitution.
- Output assembly is standardized to `implode( ', ', $results )` across multiple methods, replacing earlier `implode( ',', $results )` patterns that produced compacted, non-standard CSS output.

## Contribution

Merged by @dmsnell (co-authored by @tellthemachines). This PR synchronizes the Gutenberg repository with follow-up corrections applied in upstream WordPress Core backports (#11857, #12306), which uncovered additional trimming and fast-path updates missed during an earlier review cycle.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
