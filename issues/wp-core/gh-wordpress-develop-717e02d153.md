# HTML API: Ensure attribute value prefix matches the full search text.

- **Source:** WordPress/wordpress-develop
- **Type:** Commit
- **Author:** Jon Surrell
- **Committed:** 2026-07-13
- **Commit:** [`717e02d153`](https://github.com/WordPress/wordpress-develop/commit/717e02d153bea3f5bce1133aed130dbfd5208207)
- **Usefulness:** 4/5

## Summary

The `WP_HTML_Decoder::attribute_starts_with()` method has been fixed to correctly verify that the entire search prefix matches the decoded attribute value. Previously, the method incorrectly returned `true` if the attribute value was exhausted before the search string was fully consumed, causing false-positive prefix matches.

## Impact

- **Block & plugin developers**: Developers using the HTML API to validate or filter attribute prefixes will now see corrected match results. Logic that implicitly relied on the previous false-positive behavior may now correctly reject shorter attribute values.
- **Hosting & platform teams**: No action required. The change is internal to the HTML parser and does not alter external APIs, REST routes, or block editor interfaces.

## Technical details

The fix modifies `WP_HTML_Decoder::attribute_starts_with()` in `src/wp-includes/html-api/class-wp-html-decoder.php` to enforce full prefix consumption. Previously, the loop exited when the haystack ran out of bytes and defaulted to `return true;`, even if `$search_text` still had unmatched bytes. The return statement was changed to `return $search_at === $search_length;`, ensuring the search cursor has advanced through the entire prefix before confirming a match. The character reference comparison logic was also tightened to correctly handle partial matches within decoded tokens:

```php
$match_length = min( strlen( $next_chunk ), $search_length - $search_at );
if ( 0 !== substr_compare( $search_text, $next_chunk, $search_at, $match_length, $loose_case ) ) {
    return false;
}
$search_at += $match_length;
```

This prevents false positives when an attribute value is shorter than the search string (e.g., `'java'` no longer matches the prefix `'javascript'`). Boundary test cases were added in `tests/phpunit/tests/html-api/wpHtmlDecoder.php`.

## Contribution

Authored by Jon Surrell and merged as commit `717e02d` (PR #12380, ticket #65372). Review input was provided by dmsnell. The change isolates the boundary condition where the attribute value ends before the search prefix is fully consumed, replacing the blanket `return true` with an explicit cursor-length check and aligning cursor advancement with decoded character reference lengths.

---

*This content is AI-generated and may contain errors. See [WP Digests](https://github.com/philltran/wp-digests/) — verify against the linked upstream source. Not affiliated with or endorsed by the WordPress project, the WordPress Foundation, or Automattic.*
