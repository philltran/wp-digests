# Taxonomy: Limit term slugs to 200 characters in `wp_unique_term_slug()`.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Peter Wilson
- **Committed:** 2026-09-23
- **Commit:** [`98d218765f`](https://github.com/WordPress/wordpress-develop/commit/98d218765f16ffac15fe6c17419786a4a36b990f)
- **Usefulness:** 3/5

## Summary

`wp_unique_term_slug()` now truncates generated term slugs to the 200-character limit of the `terms.slug` column when it appends a parent slug or a numeric suffix. Previously, non-Latin term names, which are stored percent-encoded (a 21-character Cyrillic name becomes a 116-character slug), could overflow the column when combined with a parent suffix, causing the insert or update to fail. The truncation logic is extracted into a new generic `wp_truncate_slug()`, and `_truncate_post_slug()` is deprecated in its favour.

## Impact

**Plugin & theme developers**
- `_truncate_post_slug()` is deprecated as of 7.2.0. Replace it with `wp_truncate_slug()`, which has the same signature (`$slug`, `$length = 200`). The old function remains as a wrapper that calls `_deprecated_function()`, so any remaining callers will emit a deprecation notice but keep working.
- `wp_truncate_slug()` is marked `@access private` in its docblock, despite the non-underscore name. Treat it with caution as a public API.
- Code that filters `wp_unique_term_slug_is_bad_slug` or relies on the exact output of `wp_unique_term_slug()` may see different slugs, but only when the result would otherwise exceed 200 characters.

**Site owners**
- Hierarchical taxonomies with non-Latin term names (e.g. same-named child and parent terms) no longer fail to save. No action required.

**Hosting / headless**
- No schema changes and no REST changes. Slugs that were already valid are unchanged.

## Technical details

**New function** in `wp-includes/formatting.php`, `wp_truncate_slug( $slug, $length = 200 )`. It is the body of the old `_truncate_post_slug()` moved unchanged: if `strlen( $slug ) > $length`, it runs `urldecode()`; a plain ASCII slug is cut with `substr()`, while a percent-encoded slug is decoded and re-encoded through `utf8_uri_encode( $decoded, $length, true )` so a `%XX` sequence is never cut in half. The result is `rtrim( ..., '-' )`.

**`wp_unique_term_slug()`** (`wp-includes/taxonomy.php`):

```php
// Before
$slug .= $parent_suffix;
...
$alt_slug = $slug . "-$num";

// After
$slug = wp_truncate_slug( $slug . $parent_suffix, 200 );
...
$numeric_suffix = "-$num";
$alt_slug       = wp_truncate_slug( $slug, 200 - strlen( $numeric_suffix ) ) . $numeric_suffix;
```

The numeric suffix branch reserves room for the suffix on each loop iteration, so `-2` through `-10` and beyond still fit within 200 characters. A `@since 7.2.0` note was added to the docblock.

**Call-site migration:** all internal `_truncate_post_slug()` calls (three in `wp_unique_post_slug()`, one in `wp_add_trashed_suffix_to_post_name_for_post()`, one in `wp_filter_wp_template_unique_post_slug()`) now call `wp_truncate_slug()`. The old function is removed from `post.php` and re-added to `deprecated.php`.

**Tests:** new `wpTruncateSlug.php`, plus cases in `wpUniqueTermSlug.php`, `wpInsertTerm.php` and `wpUpdateTerm.php` (parent-suffix overflow, 200-char and 198-char boundary with `-2` and `-10` suffixes, and 12 colliding long slugs staying unique). The existing `truncatePostSlug.php` test now carries `@expectedDeprecated _truncate_post_slug`. One test comment notes `wp_update_term()` ignores the return value of `$wpdb->update()`, so an overlong slug previously left the whole row unwritten without an error.

## Contribution

Committed to trunk by Peter Wilson (r63909), fixing a long-standing ticket (#46010) that had many contributors credited in the props. The record carries no discussion detail beyond the commit message; the choice to extract a generic `wp_truncate_slug()` and keep the old test suite under a deprecation expectation is stated in the message.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
